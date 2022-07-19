## What is ChromeHtmlToPdf?

ChromeHtmlToPdf is a 100% managed C# .NETStandard 2.0 library and .NET Core 3.1 console application (that also works on Linux and macOS) that can be used to convert HTML to PDF format with the use of Google Chrome

## Why did I make this?

I needed a replacement for wkHtmlToPdf, a great tool but the activity on this project is low and it's not 100% compatible with HTML5.

## License Information

ChromeHtmlToPdf is Copyright (C)2017-2022 Kees van Spelde and is licensed under the MIT license:

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NON INFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.

## Installing via NuGet

The easiest way to install ChromeHtmlToPdf is via NuGet.

In Visual Studio's Package Manager Console, simply enter the following command:

    Install-Package ChromeHtmlToPdf 

### Converting a file or url from code

```csharp
using (var converter = new Converter())
{
    converter.ConvertToPdf(new Uri("http://www.google.nl"), @"c:\google.pdf");
}

// Show the PDF
System.Diagnostics.Process.Start(@"c:\google.pdf");
```

### Converting from Internet Information Services (IIS)

- Download Chrome portable and extract it
- Let your website run under the ApplicationPool identity
- Copy the files to the same location as where your project exists on the webserver
- Reference the ChromeHtmlToPdfLib.dll from your webproject
- Call the converter.ConverToPdf method from code

Thats it.

If you get strange errors when starting Chrome than this is due to the account that is used to run your site. I had a simular problem and solved it by hosting ChromeHtmlToPdf in a Windows service and making calls to it with a WCF service.

### Converting from the command line

```csharp
ChromeHtmlToPdf.exe --input https://www.google.com --output c:\google.pdf
```
![screenshot](https://github.com/Sicos1977/ChromeHtmlToPdf/blob/master/console.png)

### Console app exit codes

0 = successful, 1 = an error occurred

Installing on Linux or macOS
============================

### Installing .NET

See this url about how to install .NET on Linux

https://docs.microsoft.com/en-us/dotnet/core/install/linux

And this url about how to install .NET on macOS

https://docs.microsoft.com/en-us/dotnet/core/install/macos

### Installing Chrome

See this url about how to install Chrome on Linux

https://support.google.com/chrome/a/answer/9025903?hl=en

And this url about how to install Chrome on macOS

https://support.google.com/chrome/a/answer/7550274?hl=en

### Example installing Chrome on Linux Ubuntu

```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'

sudo apt-get update

sudo apt-get install google-chrome-stable

google-chrome --version

google-chrome --no-sandbox --user-data-dir
```

Pre compiled binaries
=====================

You can find pre compiled binaries for Windows, Linux and macOS over here 

Latest version
---------------
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.6.0/ChromeHtmlToPDF_260.zip

Older versions
--------------
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.0.11/ChromeHtmlToPdf_211.zip
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.1.6/ChromeHtmlToPdf_216.zip
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.2/ChromeHtmlToPdf_220.zip
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.5.1/ChromeHtmlToPdf_251.zip
https://github.com/Sicos1977/ChromeHtmlToPdf/releases/download/2.5.33/ChromeHtmlToPdf_253.zip

.NET Core 3.1 for the console app
---------------------------------
The console app needs .NET Core 3.1 to run, you can download this framework from here 

https://dotnet.microsoft.com/en-us/download/dotnet/3.1

Logging
=======

From version 2.5.0 ChromeHtmlToPdfLib uses the Microsoft ILogger interface (https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0). You can use any logging library that uses this interface.

ChromeHtmlToPdfLib has some build in loggers that can be found in the ```ChromeHtmlToPdfLib.Logger``` namespace. 

For example

```csharp
var logger = !string.IsNullOrWhiteSpace(<some logfile>)
                ? new ChromeHtmlToPdfLib.Loggers.Stream(File.OpenWrite(<some logfile>))
                : new ChromeHtmlToPdfLib.Loggers.Console();
```

Setting a common Chrome cache directory
=======================================

You can not share a cache directory between Chrome instances because the first instance that is using the cache directory will lock it for its own use. The most efficient way to make optimal use of a cache directory is to create one for each instance that you are running. 

I'm using Chrome from a WCF service and used the class below to make optimal use of cache directories. The class will create an instance id that I use to create a cache directory for each running Chrome instance. When the instance shuts down the instance id is put back in a stack so that the next executing instance can use this directory again.

```csharp
public static class InstanceId
{
    #region Fields
    private static readonly ConcurrentStack<string> ConcurrentStack;
    #endregion

    #region Constructor
    static InstanceId()
    {
        ConcurrentStack = new ConcurrentStack<string>();

        for(var i = 100000; i > 0; i--)
            ConcurrentStack.Push(i.ToString().PadLeft(6, '0'));
    }
    #endregion

    #region Pop
    /// <summary>
    /// Returns an instance id and pops it from the <see cref="ConcurrentStack"/>
    /// </summary>
    /// <returns></returns>
    public static string Pop()
    {
        if (ConcurrentStack.TryPop(out var instanceId))
            return instanceId;

        throw new Exception("Instance id stack is empty");
    }
    #endregion

    #region Push
    /// <summary>
    /// Pushes the <paramref name="instanceId"/> back on top of the <see cref="ConcurrentStack"/>
    /// </summary>
    /// <param name="instanceId"></param>
    public static void Push(string instanceId)
    {
        ConcurrentStack.Push(instanceId);
    }
    #endregion
}
```

Using it in a Docker container
==============================

```
# Suppress an apt-key warning about standard out not being a terminal. Use in this script is safe.
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn

# export DEBIAN_FRONTEND="noninteractive"
ENV DEBIAN_FRONTEND noninteractive

# Install deps + add Chrome Stable + purge all the things
RUN apt-get update && apt-get install -y \
	apt-transport-https \
	ca-certificates \
	curl \
	gnupg \
	--no-install-recommends \
	&& curl -sSL https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
	&& echo "deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list \
	&& apt-get update && apt-get install -y \
	google-chrome-stable \
	--no-install-recommends \
	&& apt-get purge --auto-remove -y curl gnupg \
	&& rm -rf /var/lib/apt/lists/*

# Chrome Driver
RUN apt-get update && \
    apt-get install -y unzip && \
    wget https://chromedriver.storage.googleapis.com/2.31/chromedriver_linux64.zip && \
    unzip chromedriver_linux64.zip && \
    mv chromedriver /usr/bin && rm -f chromedriver_linux64.zip
```

See this issue for more information --> https://github.com/Sicos1977/ChromeHtmlToPdf/issues/39

# When using it on Linux

The code will not work when Chrome is started in sandbox mode (default) on Linux. If you get an error you can't explain then add this to your code and try again.

```csharp
converter.AddChromeArgument("--no-sandbox")
```

Core Team
=========
    Sicos1977 (Kees van Spelde)

Support
=======
If you like my work then please consider a donation as a thank you.

<a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=NS92EXB2RDPYA" target="_blank"><img src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif" /></a>

## Reporting Bugs

Have a bug or a feature request? [Please open a new issue](https://github.com/Sicos1977/ChromeHtmlToPdf/issues).

Before opening a new issue, please search for existing issues to avoid submitting duplicates.
