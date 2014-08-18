---
layout: post
title: WPF != Fancy Winforms
tags: [c#,dot-net,wpf]
alias: /2011/02/wpf-fancy-winforms.html
---

I'm a big fan of Stackoverflow, and I was just recently looking at this 
<a href="http://stackoverflow.com/questions/4965023/c-check-if-text-file-has-content">very simple question regarding changing button color.</a> 
On the surface it seems this would warrant a very simple answer, and indeed 
<a href="http://stackoverflow.com/questions/4965023/c-check-if-text-file-has-content/4965058#4965058">Chris' Answer</a> 
was simple, straightforward and to the point.

However, it feels too much like how I would have done things back in the dark ages of WinForms.
There seems to be this misconception that WPF is just a fancier version of the old Win32 standby.
*(note I am not trying to imply this is what Chris thinks, it merely serves as a convenient backdrop for this post)*

But WPF is not even a cousin, or distant relative of WinForms. If WPF saw WinForms in the hallway it would beat him up
and take his lunch money from him. Anyway, that is enough of a rant. On with what started out as my original answer to
the question, which turned into this epic post...

###Original Answer Starts Here:

So Chris' suggestion is pretty straightforward, and it is certainly easier to implement than what I am about to show you ;)
However, if you are going to venture into the world of WPF then you need to experience what it has to offer.

In my opinion the two biggest advantages in WPF are:

1.  **The power and flexibility of the declarative model**
2.  **An incredibly rich data binding infrastructure**

Granted those are immensely deep topics within WPF, but without at least gaining a minimal understanding of those topics,
it will be like driving a Lamborghini around in first gear all the time.

That being said, you can jump to the full solution <a href="#fullsolution">here</a>, copy and paste it into a
clean solution, and play with it for a second. I'll be waiting patiently here to give you a detailed explanation of what is going on.

Ready? Ok then, lets get going.

Lets start with the core logic of the application, your requirements are:

1.  **Examine a file on disk**
2.  **Change the appearance of a button based on content length**
3.  **Update the appearance of the button when the file changes**

Since the appearance of the button is presentation specific, we simply won't worry about that when implementing our
core logic. So what we really need to do is monitor the file for changes, and update a local variable with the
length of the file, if it exists.

This is a perfect job for a <a href="http://msdn.microsoft.com/en-us/library/system.io.filesystemwatcher.aspx"><span class="Apple-style-span" style="font-family: 'Courier New', Courier, monospace;">FileSystemWatcher</span></a>.
The file watcher kicks off a thread in the background which utilizes some low level Win32 API's to monitor
events on the file system. We can tie into those events via the watcher, and handle them in our application asynchonously.

Creating a new watcher is pretty simple and all we really need to provide is the Path. Here I am simply using the
current location of the Executing Assembly for the base path. The second thing you see here is the Filter property.
This works just like the filter you would use in the command line.

```csharp
private FileSystemWatcher _fileWatcher;
private const String FileToWatch = "tempfile.txt";

public void InitializeFileWatcher()
{
    _fileWatcher = new FileSystemWatcher();
    _fileWatcher.Path = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
    _fileWatcher.Filter = "*.txt";

    //Snip...
}
```

Next we want to subscribe to the different types of events available, and tell the watcher to start doing it's thing.
In this case our event handler is just a thin wrapper around a simple method that does all the "heavy" lifting. Lastly
we call our `UpdateFileLength` method directly because we need to know what the status of the file is right now,
 without waiting on an event from our watcher.

```csharp
public void InitializeFileWatcher()
{
    //Snip...

    _fileWatcher.Created += fileWatcher_Handler;
    _fileWatcher.Changed += fileWatcher_Handler;
    _fileWatcher.Deleted += fileWatcher_Handler;
    _fileWatcher.Renamed += fileWatcher_Handler;

    _fileWatcher.EnableRaisingEvents = true;

    UpdateFileLength(Path.Combine(_fileWatcher.Path, FileToWatch));
}

private void UpdateFileLength(string filePath)
{
    Int64? length = null;

    if (Path.GetFileName(filePath) == FileToWatch)
    {
        var fi = new FileInfo(filePath);

        if (fi.Exists)
            length = fi.Length;
    }

    Dispatcher.Invoke((Action)(
        () => FileContentLength = length)
    );
}

void fileWatcher_Handler(object sender, FileSystemEventArgs e)
{
    UpdateFileLength(e.FullPath);
}
```

The UpdateFileLength method is pretty simple. It checks to see if the file is the one we are looking for, extracts the
length if it exists, and updates a property on our MainWindow. Something that may seem a bit odd however,
is the use of the <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatcher.aspx">`Dispatcher`</a> and an anonymous method in order to set that property.

Remember how I said that the `FileSystemWatcher` kicks off a background thread. Well, technically `UpdateFileLength`
could be getting executed from a different thread than the UI thread. And well... it's a big NO NO to access things
in the UI from another thread. Without going into a a lot of detail, the Dispatcher is responsible for scheduling
things to happen on the UI thread in WPF. Here we are ensuring that no matter what thread executes this method,
that that line will always be executed on the UI thread.

Ok, so the last little bit is our property we use to store the current length of the file we are monitoring, but it is no ordinary property.

```csharp
public static readonly DependencyProperty FileContentLengthProperty = DependencyProperty.Register(
    "FileContentLength",
    typeof(Int64?),
    typeof(MainWindow),
    new UIPropertyMetadata(null));

public Int64? FileContentLength
{
    get
    {
        return (Int64?)GetValue(FileContentLengthProperty);
    }
    set
    {
        SetValue(FileContentLengthProperty, value);
    }
}
```

What you are staring at here is known as a <a href="http://msdn.microsoft.com/en-us/library/ms752914.aspx">`DependencyProperty`</a>,
and they are pretty foundational to how WPF works, so take some time to learn about them. For our purposes, just
know that what this provides for us is a way for WPF to monitor changes to our property, which allows for all sorts of fun stuff in the UI.

Now, on to the UI and where the real magic happens. Did you notice we didn't ever reference any buttons or set colors in
the code behind? That's because 99% of the time in WPF there is no need to. The declarative model we have in
<a href="http://msdn.microsoft.com/en-us/library/ms752059.aspx">XAML</a> combined with the rich data binding ecosystem
in WPF gives us the freedom to finally have true seperation of concerns between presentation and logic.

The first order of business is to set the
<a href="http://msdn.microsoft.com/en-us/library/system.windows.frameworkelement.datacontext.aspx">`DataContext`</a> on the
Window itself so that we can access all of the properties defined in our code behind. Ok there is only one, but we still need it.

```csharp
DataContext="{Binding RelativeSource={RelativeSource Self}}"
```

Next we need a button on our window. Ok, nothing special going on here, except that there is no content, no coloring,
just a Style property bound to some static resource called `"FileBoundButton"`

```xml
<button Style="{StaticResource FileBoundButton}" Name="button1" />
```

<a href="http://msdn.microsoft.com/en-us/library/ms745683.aspx">Styles in WPF are another big complex topic</a>, but
just know that this isn't anything like CSS. You can set virtually any property on an element via a style, and include
declarative triggers to change the appearance based on events, data, or even other elements. This is precisely how we
are going to achieve the visual affect you are after and then some!

```xml
<Style x:Key="FileBoundButton" TargetType="ContentControl">
    <Setter Property="Background" Value="Green" />
    <Setter Property="Content" Value="{Binding FileContentLength}" />
    <Style.Triggers>
        <Datatrigger Binding="{Binding FileContentLength}" Value="{x:Null}">
            <setter Property="Background" Value="LightGray" />
            <setter Property="Content" Value="No File!" />
        </DataTrigger>
        <Datatrigger Binding="{Binding FileContentLength}" Value="0">
            <Setter Property="Background" Value="Red" />
        </DataTrigger>
    </Style.Triggers>
</Style>
```

The style we are using is declared as a static resource on the window itself. You need a name if you want to be able to
refer to it later, and using a TargetType of ContentControl instead of a Button means we could reuse this style a lot
of different elements, not just a button.

We set the Background property to Green initially, and we also set the Content equal to the value of our
`FileContentLength` property. Then we make use of DataTriggers to alter those settings based on the value of
FileContentLength. The syntax of a DataTrigger can be a bit wonky at first, but if you were to spell it
out in pseudo-code it would sound something like this:

<pre>
set the Background property to <span class="Apple-style-span" style="background-color: lime;">Green</span>
set the Content to DataContext.FileContentLength

<span class="Apple-style-span" style="color: blue;">IF</span> the value of DataContext.FileContentLength is <span class="Apple-style-span" style="color: red;">NULL</span>
    <span class="Apple-style-span" style="color: blue;">THEN</span> set the Background property to <span class="Apple-style-span" style="background-color: #cccccc;">LightGray</span> <span class="Apple-style-span" style="color: blue;">AND</span>
         set the Content to "No File!"
<span class="Apple-style-span" style="color: blue;">ELSE IF</span> the value of DataContext.FileContentLength is <span class="Apple-style-span" style="color: red;">0</span>
    <span class="Apple-style-span" style="color: blue;">THEN</span> set the Background property to <span class="Apple-style-span" style="background-color: red;">Red</span>
</pre>

You might be wondering exactly <i>what</i> DataContext this is referring to. Essentially whatever framework element
this style happens to be applied to. Because we didn't set the Button's DataContext explicitely it will inherit the
DataContext of the Window. But be careful! Where the DataContext comes from isn't always apparent.

Now, if you put all this together and launch the app for the first time you should end up with a small
window with a gray button in it that says "No File!"

<div style="text-align: center;"><img src="https://lh6.googleusercontent.com/_zrLYGpcNAqw/TVTfmg2jvoI/AAAAAAAAA18/PrNnn4ucwoc/s800/FileWatcher_NoFile.png" /></div>

Without closing the program, navigate to the executing directory of your application "[ProjectFolder]\bin\Debug" and
create a new text file named "tempfile.txt" Now you should see the button change color to red and display a "0" since the file is empty.

<div style="text-align: center;"><img src="https://lh4.googleusercontent.com/_zrLYGpcNAqw/TVTfmYRStiI/AAAAAAAAA10/pBNoBbSkEc0/s800/FileWatcher_EmptyFile.png" /></div>

Open the newly created text file and write something in there. Save the file and again, watch as your button magically
changes before your eyes to green, and displaying "N" where N is the number of characters in your text file.

<div style="text-align: center;"><img src="https://lh3.googleusercontent.com/_zrLYGpcNAqw/TVTfmQqJ5tI/AAAAAAAAA14/7jZ_skxgL4o/s800/FileWatcher_FileWithContent.png" /></div>

If you actually made it through this whole thing, then you have hopefully been opened up to a whole new world of
possibilities by using WPF. Just remember that it is a very, very deep technology and you won't learn it over night.
Here are some resources to get you started on your journey though.

 - <a href="http://msdn.microsoft.com/en-us/library/ms754130.aspx">The MSDN Walk Through.</a> Heavy reading, but worth it.
 - <a href="http://www.wpftutorial.net/christian.php">Christian Moser's WPF Tutorial Site.</a> Lots of easy to digest samples.
 - <a href="http://stackoverflow.com/questions/tagged/wpf">WPF Questions on Stackoverflow</a>

<a name="fullsolution"></a>

`MainWindow.xaml`:

```xml
<Window x:Class="TechnoFattie.WPF.MainWindow" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" mc:Ignorable="d" Height="86" Width="174"
        Title="File Watcher Example" DataContext="{Binding RelativeSource={RelativeSource Self}}">
    <Window.Resources>
        <Style x:Key="FileBoundButton" TargetType="Control">
            <setter Property="Background" Value="Green"/>
            <Style.Triggers>
            <DataTrigger Binding="{Binding FileContentLength}" Value="{x:Null}">
            <setter Property="Background" Value="LightGray"/>
            </DataTrigger>
              <DataTrigger Binding="{Binding FileContentLength}" Value="0">
            <setter Property="Background" Value="Red"/>
            </DataTrigger>
              </Style.Triggers>
        </Style>
    </Window.Resources>
    <Grid x:Name="LayoutRoot">
        <button Content="{Binding FileContentLength}" Style="{StaticResource FileBoundButton}" Name="button1"/>
    </Grid>
</Window>
```

`MainWindow.xaml.cs`:

```csharp
public partial class MainWindow : Window
{
    private FileSystemWatcher _fileWatcher;
    private const String FileToWatch = "tempfile.txt";

    public MainWindow()
    {
        InitializeComponent();
            
        InitializeFileWatcher();
    }

    public void InitializeFileWatcher()
    {
        _fileWatcher = new FileSystemWatcher();
        _fileWatcher.Path = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
        _fileWatcher.Filter = "*.txt";

        _fileWatcher.Created += fileWatcher_Handler;
        _fileWatcher.Changed += fileWatcher_Handler;
        _fileWatcher.Deleted += fileWatcher_Handler;
        _fileWatcher.Renamed += fileWatcher_Handler;

        _fileWatcher.EnableRaisingEvents = true;

        UpdateFileLength(Path.Combine(_fileWatcher.Path, FileToWatch));
    }

    private void UpdateFileLength(string filePath)
    {
        Int64? length = null;

        if (Path.GetFileName(filePath) == FileToWatch)
        {
            var fi = new FileInfo(filePath);

            if (fi.Exists)
                length = fi.Length;
        }

        Dispatcher.Invoke((Action)(() => FileContentLength = length));
    }

    void fileWatcher_Handler(object sender, FileSystemEventArgs e)
    {
        UpdateFileLength(e.FullPath);
    }

    public Int64? FileContentLength
    {
        get
        {
            return (Int64?)GetValue(FileContentLengthProperty);
        }
        set
        {
            SetValue(FileContentLengthProperty, value);
        }
    }

    public static readonly DependencyProperty FileContentLengthProperty = DependencyProperty.Register(
        "FileContentLength",
        typeof(Int64?),
        typeof(MainWindow),
        new UIPropertyMetadata(null));
}
```
