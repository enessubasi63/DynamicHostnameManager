Tabii, aşağıda bu program için daha detaylı bir README dosyası örneği bulunmaktadır:

```markdown
# AutoHostnameUpdater

## Overview

AutoHostnameUpdater is a C# application designed to automate the process of changing the hostname of a computer based on a predefined mapping stored in a text file. The program reads the current hostname of the machine, compares it with the mappings in the text file, and if a new hostname is found, it updates the machine's hostname accordingly. Additionally, the application logs all operations and errors to a log file for easy troubleshooting and auditing.

## Key Features

- **Hostname Mapping**: Reads a text file containing hostname mappings in the format `current_hostname,new_hostname`.
- **Hostname Update**: Updates the machine's hostname if a new hostname is found in the mapping.
- **Logging**: Logs all activities, including reading the hostname map, updating the hostname, and any errors encountered during the process.

## Requirements

- .NET Framework 4.7.2 or higher
- Administrative privileges to run the application and change the hostname
- PowerShell installed on the target machine

## How It Works

### Initial Setup

- **File Paths**: The program expects the hostname mapping file to be located at `C:\Path\To\Your\Hostname.txt`. The log file is created at `C:\Logs\AppLog.txt`. You can modify these paths in the code to suit your environment.
- **Credentials**: The program uses domain credentials to authenticate and update the hostname. These should be replaced with your actual domain username and password in the `ChangeComputerName` method.

### Reading the Hostname Map

The `ReadHostnameMap` method reads the hostname mapping file, which should contain lines formatted as `current_hostname,new_hostname`. It parses these lines into a dictionary for easy lookup.

### Updating the Hostname

The `ChangeComputerName` method uses PowerShell commands to update the computer's hostname. It constructs a PowerShell script that authenticates using the provided domain credentials and then changes the hostname. Any errors encountered during this process are logged.

### Logging

All operations are logged to `C:\Logs\AppLog.txt`. This includes successful reads of the hostname map, successful hostname updates, and any errors.

## Code Explanation

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Management.Automation;
using System.Net;
using System.Security;

class Program
{
    static void Main(string[] args)
    {
        string filePath = @"C:\Path\To\Your\Hostname.txt"; // Update with your file path
        string logPath = Path.Combine("C:\\Logs", "AppLog.txt"); // Update with your desired log path
        string currentHostname = Environment.MachineName;

        using (StreamWriter logWriter = new StreamWriter(logPath, true))
        {
            try
            {
                if (!File.Exists(filePath))
                {
                    throw new FileNotFoundException($"File not found: {filePath}");
                }

                var hostnameMap = ReadHostnameMap(filePath);
                logWriter.WriteLine($"{DateTime.Now} - Hostname map read.");

                if (hostnameMap.TryGetValue(currentHostname, out var newHostname))
                {
                    ChangeComputerName(newHostname, logWriter);
                }
                else
                {
                    logWriter.WriteLine($"{DateTime.Now} - New hostname not found or already current.");
                }
            }
            catch (Exception ex)
            {
                logWriter.WriteLine($"{DateTime.Now} - Error: {ex.Message}");
            }
        }
    }

    private static Dictionary<string, string> ReadHostnameMap(string filePath)
    {
        var map = new Dictionary<string, string>();
        var lines = File.ReadAllLines(filePath);
        foreach (var line in lines)
        {
            var parts = line.Split(',');
            if (parts.Length == 2)
            {
                map[parts[0].Trim()] = parts[1].Trim();
            }
        }
        return map;
    }

    private static void ChangeComputerName(string newHostname, StreamWriter logWriter)
    {
        string domainUsername = "your_domain_username"; // Replace with your domain username
        string domainPassword = "your_domain_password"; // Replace with your domain password

        SecureString securePassword = new NetworkCredential("", domainPassword).SecurePassword;

        PowerShell ps = PowerShell.Create();
        ps.AddScript($"$credential = New-Object System.Management.Automation.PSCredential " +
                     $"('{domainUsername}', (ConvertTo-SecureString '{domainPassword}' -AsPlainText -Force)); " +
                     $"Rename-Computer -NewName {newHostname} -DomainCredential $credential -Force");

        var results = ps.Invoke();

        if (ps.Streams.Error.Count > 0)
        {
            logWriter.WriteLine($"{DateTime.Now} - Computer name change failed.");
            foreach (var error in ps.Streams.Error)
            {
                logWriter.WriteLine(error.ToString());
            }
        }
        else
        {
            logWriter.WriteLine($"{DateTime.Now} - Computer name successfully changed to: " + newHostname);
        }
    }
}
```

### Usage Instructions

1. **Clone the Repository**: Clone the repository to your local machine.
    ```sh
    git clone https://github.com/yourusername/AutoHostnameUpdater.git
    ```
2. **Update File Paths**: Modify the `filePath` and `logPath` variables in the code to point to your actual hostname mapping file and desired log file location.
3. **Update Credentials**: Replace the placeholder domain username and password with your actual domain credentials in the `ChangeComputerName` method.
4. **Compile and Run**: Compile the application using your preferred IDE or command-line tools and run it with administrative privileges.

### Example Hostname Mapping File

The hostname mapping file should be a plain text file with each line containing a current hostname and the corresponding new hostname, separated by a comma. For example:

```
OLD-HOSTNAME1,NEW-HOSTNAME1
OLD-HOSTNAME2,NEW-HOSTNAME2
OLD-HOSTNAME3,NEW-HOSTNAME3
```

### Troubleshooting

- **File Not Found**: Ensure the path to the hostname mapping file is correct.
- **Permissions**: Run the application with administrative privileges to allow hostname changes.
- **Logging**: Check the log file at `C:\Logs\AppLog.txt` for detailed error messages and operation logs.

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your changes. Ensure your code adheres to the existing coding standards and includes appropriate tests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgements

Special thanks to all contributors and the open-source community for their invaluable support and contributions.

```

This README provides a comprehensive guide to understanding, setting up, and using the AutoHostnameUpdater application. If you need further customization or additional sections, please let me know!
