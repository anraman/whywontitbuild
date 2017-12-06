# Using Azure IoT Hub Device Twins with Xamarin.Forms Apps

## **What Didn't Build?**

[Azure IoT Hub Device Twins](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-device-twins) are useful for monitoring and updating the state of the devices connected to an IoT Hub (amongst other things - see the [docs](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-device-twins) for details). There are many reasons you may want to make use of them in your IoT solution, however if you are planning on accessing/manipulating Device Twin details using a Xamarin.Forms application, there are a few additional steps required before this will work.

## **TL;DR**

You can use Device Twins from a Xamarin.Forms application by dropping the ```TransportType.Mqtt``` in the DeviceTwin constructor and adding some extra Nugets.

>Note: The solution described below will not work for PCL-based Xamarin.Forms projects as MQTT is not supported in PCL projects. [Migration to .NET Standard](https://blogs.msdn.microsoft.com/premier_developer/2017/10/27/converting-pcl-portable-class-libraries-libraries-to-net-standard-class-libraries/) is recommended.

## **Solution Description**

It is possible to use Device Twins from a Xamarin.Forms app with the following tweaks. The [GitHub repo](https://github.com/anraman/XamarinIoT) contains working samples for shared project and .NET Standard-based Xamarin.Forms apps, as well as some console apps for the testing and configuration of Device Twins. All the important code is located in ```MainPage.xaml.cs``` for convenience.

The .NET Standard project ([XamarinIoTStandard](https://github.com/anraman/XamarinIoT/tree/master/XamarinIoTStandard)) is a conversion from a PCL project, not one created directly using the VS 2017 Preview Xamarin tools.

The sample code provided does the following:

1. Connects to the specified IoT Hub.
2. Retrieves the relevant Device Twin.
3. Sets the telemetry reporting frequency to '24h'.
4. Listens for config updates from the IoT Hub (you can use the [ConsoleIoTService](https://github.com/anraman/XamarinIoT/tree/master/ConsoleIoTService) project included to set the desired state as described [here](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-csharp-csharp-twin-how-to-configure#create-the-service-app)).
5. When an update to the desired state is detected, the ```OnDesiredPropertyChanged``` method is triggered, updating the Device Twin as necessary. If you monitor the Device Twin (using the [Azure Portal](https://portal.azure.com)) during this operation, you will see it update a few times and go through a ‘pending’ phase as the device ‘resets’ before reporting the desired value ('5m').

All samples are based on the following two tutorials (with modifications made for compatibility with Xamarin.Forms):

- [Get started with device twins (.NET/.NET)](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-csharp-csharp-twin-getstarted)

- [Use desired properties to configure devices](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-csharp-csharp-twin-how-to-configure)

This solution has been tested on UWP (physical devices), Android (physical device) and iOS (simulator).

## **The Specifics**

### UWP Project

No changes required. [Tutorial code](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-csharp-csharp-twin-getstarted) works as expected using MQTT to communicate with the IoT Hub.

### Android/iOS Projects

Work only if you *omit* ```TransportType.MQTT``` from the ```DeviceClient``` constructor. Both projects also require the manual addition of several Nuget packages. Remediation steps are as follows:

1. Use the following code when creating the IoT Hub ```DeviceClient```:

    ```csharp
    if (Device.RuntimePlatform == Device.Android || Device.RuntimePlatform == Device.iOS)
    {
        Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString);
    }
    else
    {
        Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, TransportType.Mqtt);
    }
    ```

2. Add the following Nuget packages (correct at the time of writing):
    
    - ```Newtonsoft.Json (latest stable (10.x))```
    - ```Microsoft.Azure.Devices.Shared (1.2.0-preview-001 or below)```
    - ```WindowsAzure.Storage (8.5.0)```
    - ```DotNetty.Transport (0.4.7)```
    - ```DotNetty.Codecs.Mqtt (0.4.7)```
    - ```DotNetty.Handlers (0.4.7)```
    
    In the case of iOS, ```Microsoft.Azure.Amqp (2.1.2)``` is also required. The packages required in your case may vary but the errors should guide you to which ones are missing. This is a [known issue](https://github.com/Azure/azure-iot-sdk-csharp/issues/158).

3. Clean & rebuild the solution to remove erroneous errors. You may see a persistent error complaining that 'InitializeComponent does not exist in the current context' - this is usually safe to ignore and shouldn't stop the project from building (try restarting Visual Studio/the computer if this is giving you trouble).