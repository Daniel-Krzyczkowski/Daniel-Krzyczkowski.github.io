---
title: "Real-time data with Microsoft Azure SignalR Service"
---

<p align="center">
<img src="/images/devisland/article7/assets/signalrservicelogo.png?raw=true" alt="Real-time data with Microsoft Azure SignalR Service"/>
</p>

<h3><strong>Short introduction</strong></h3>
There are many application types that require real-time content updates. GPS live updates, score updates in games or chat messages. In this special cases Microsoft Azure SignalR Service can help. SignalR is an abstraction over a number of techniques used for building real-time web applications. WebSockets, Server-Sent Events (SSE) and Long Polling are used interchangeably when other are unavailable. In this article I would like to present my Proof of Concept (PoC) how to use Azure SignalR Service with Univeral Windows Application (UWP) to display real-time data about transport (similar to Uber map). Please note that at the moment of writing this post Azure SignalR Service is in preview.
<p style="text-align:center;"><img class="alignnone wp-image-631" src="/images/devisland/article7/assets/signalrdiagram.png?w=197" alt="" width="246" height="375" /></p>
<img class=" wp-image-575 aligncenter" src="/images/devisland/article7/assets/signalrservice6.png?w=300" alt="" width="429" height="339" />
<h3><strong>Solution structure</strong></h3>
To simulate real-time transport data I prepared three applications for this solution:
<ul>
 	<li>ASP .NET Core Web API - web API integrated with Azure SignalR service to send rea-time transport data to client app</li>
 	<li>Console App - created to send fake location data to Web API about transport</li>
 	<li>Universal Windows App - client application to display information on map about drivers</li>
</ul>
<h3><strong>Setup Azure SignalR Service</strong></h3>
Open <a href="http://portal.azure.com" target="_blank" rel="noopener">Microsoft Azure</a> portal. Type "SignalR Service" in search window:

<img class=" wp-image-580 aligncenter" src="/images/devisland/article7/assets/signalrservice1.png?w=300" alt="" width="478" height="180" />

Fill required information like name of the service (used later with full URL):

<img class=" wp-image-583 aligncenter" src="/images/devisland/article7/assets/signalrservice2.png?w=178" alt="" width="239" height="403" />

Once service is created open the tab and select "Keys" section:

<img class=" wp-image-585 aligncenter" src="/images/devisland/article7/assets/signalrservice3.png?w=300" alt="" width="463" height="270" />
<p style="text-align:center;"><img class="alignnone wp-image-586" src="/images/devisland/article7/assets/signalrservice4.png?w=300" alt="" width="463" height="171" /></p>
Copy Primary Connection String because we will use it later in Web API application setup.
<h3><strong>Setup ASP .NET Core Web API</strong></h3>
Create new ASP .NET Core Web API application in Visual Studio.
<p style="text-align:center;"><img class="alignnone wp-image-591" src="/images/devisland/article7/assets/signalrservice7.png?w=300" alt="" width="381" height="268" /></p>
Add "Microsoft.Azure.SignalR" NuGet package. Please note that at the moment of writing this article its in preview:

<img class=" wp-image-593 aligncenter" src="/images/devisland/article7/assets/signalrservice5.png?w=300" alt="" width="412" height="151" />

Add two solution folders: "Hubs" and "Model":
<p style="text-align:center;"><img class="alignnone wp-image-597" src="/images/devisland/article7/assets/signalrservice8.png?w=198" alt="" width="291" height="441" /></p>
Inside "Hubs" folder add new class called "TransportHub". Inside "Model" folder create "LocationUpdate" class.

"TransportHub" class is responsible for handling messages from clients and broadcasting them to all connected clients:

```
    public class TransportHub : Hub
    {
        /// <summary>
        /// Handle location update message and broadcast it to all connected clients.
        /// </summary>
        /// <param name="locationUpdate"></param>
        public void BroadcastMessage(LocationUpdate locationUpdate)
        {
            Clients.All.SendAsync("broadcastMessage", locationUpdate);
        }
    }
```

"LocationUpdate" class looks like below. "DriverName" field will be displayed in UWP application:

```
    public class LocationUpdate
    {
        public string DriverName;
        public double Latitude;
        public double Longitude;
    }
```

Now in "appsettings.json" file create new section called "AzureSignalR" and add "ConnectionString" property:

```
  "AzureSignalR":
  {
    "ConnectionString": "<<Connection string from Azure portal>>"
  }
```

Last step is to setup Azure SignalR Service in "Startup" class.

"ConfigureServices" method should look like below. We need to add SignalR Service with configuration from "appsettings.json" file:

```
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
            services.AddSignalR().AddAzureSignalR(Configuration["AzureSignalR:ConnectionString"]);
        }
```

"Configure" method should look like below. We have to map endpoint to enable communication with "TransportHub" from clients:

```
       public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            app.UseMvc();

            app.UseFileServer();
            app.UseAzureSignalR(routes =>
            {
                routes.MapHub<TransportHub>("/transport");
            });
        }
```

I published Web API source code on my GitHub so you can review the code <a href="https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/AzureSignalRTransportApp" target="_blank" rel="noopener">here.</a>
<h3><strong>Setup .NET Core Console App</strong></h3>
Now it is time to configure .NET Core Console application which will simulate location changes of two different vehicles. In the real case this console application could be replaced with IoT device or GPS application which broadcast location changes related with vehicle.
<p style="text-align:center;"><img class="alignnone wp-image-616" src="/images/devisland/article7/assets/signalrservice9.png?w=300" alt="" width="485" height="289" /></p>
Add "Microsoft.AspNetCore.SignalR.Client" NuGet. Please note that at the moment of writing this article this package is in pre-release version:
<p style="text-align:center;"><img class="alignnone wp-image-654" src="/images/devisland/article7/assets/signalrservice12.png?w=300" alt="" width="483" height="127" /></p>
Please add new class called "ClientSignalR". This class will be responsible for connecting with "TransportHub", sending messages to hub and receiving them from the hub:

```
 public class ClientSignalR
    {
        private HubConnection _hub;
        public HubConnection Hub
        {
            get
            {
                return _hub;
            }
        }

        private string _connectionUrl;
        public string ConnectionUrl
        {
            get
            {
                return _connectionUrl;
            }
        }

        public ClientSignalR()
        {
        }

        /// <summary>
        /// Initialize connection with hub.
        /// </summary>
        /// <param name="connectionUrl"></param>
        /// <returns></returns>
        public async Task Initialize(string connectionUrl)
        {
            _connectionUrl = connectionUrl;

            _hub = new HubConnectionBuilder()
                .WithUrl(_connectionUrl)
                .Build();

            await _hub.StartAsync();
        }

        /// <summary>
        /// Subscribe to receive updates from the hub.
        /// </summary>
        /// <param name="methodName"></param>
        public void SubscribeHubMethod(string methodName)
        {
            _hub.On<LocationUpdate>(methodName, (locationUpdate) =>
            {
                OnMessageReceived?.Invoke(locationUpdate);
            });
        }

        /// <summary>
        /// Send new message to hub.
        /// </summary>
        /// <param name="methodName"></param>
        /// <param name="locationUpdate"></param>
        /// <returns></returns>
        public async Task SendHubMessage(string methodName, LocationUpdate locationUpdate)
        {
            await _hub?.InvokeAsync(methodName, locationUpdate);
        }

        /// <summary>
        /// Close connection between client and hub.
        /// </summary>
        /// <returns></returns>
        public async Task CloseConnection()
        {
            await _hub.DisposeAsync();
        }

        public event Action<LocationUpdate> OnMessageReceived;
    }
```

Add another class called "LocationUpdate" - the same like in Web API project:

```
    public class LocationUpdate
    {
        public string DriverName;
        public double Latitude = 52.2326;
        public double Longitude = 20.7810;
    }
```

Now "Program" class should look like below. New ClientSignalR instance is created. I used RX Extensions here to send new location update after each two seconds. Initialize method requires URL under which SignalR Hub is available. After two seconds new location is generated for two fake drivers:

```
 class Program
    {
        static void Main(string[] args)
        {
            var hubClient = new ClientSignalR();
            hubClient.Initialize("http://localhost:63369/transport");
            var locationUpdate = new LocationUpdate()
            {
                DriverName = "Daniel"
            };

            var locationUpdate2 = new LocationUpdate()
            {
                DriverName = "Adam",
                Latitude = 52.28107,
                Longitude = 20.9558243
            };

            Observable
            .Interval(TimeSpan.FromSeconds(2))
            .Subscribe(
                async x =>
                {
                    await hubClient.SendHubMessage("broadcastMessage", locationUpdate);
                    Console.WriteLine("SENDING LOCATION UPDATE: " + locationUpdate.DriverName + " " + locationUpdate.Latitude + " " + locationUpdate.Longitude);
                    await hubClient.SendHubMessage("broadcastMessage", locationUpdate2);
                    Console.WriteLine("SENDING LOCATION UPDATE: " + locationUpdate2.DriverName + " " + locationUpdate2.Latitude + " " + locationUpdate2.Longitude);


                    locationUpdate.Latitude = locationUpdate.Latitude + 0.0008;
                    locationUpdate.Longitude = locationUpdate.Longitude + 0.0008;

                    locationUpdate2.Latitude = locationUpdate2.Latitude + 0.0008;
                    locationUpdate2.Longitude = locationUpdate2.Longitude + 0.0008;
                });

            Console.ReadKey();
        }
    }
```

I published Console app source code on my GitHub <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/TransportAppSimulator" target="_blank" rel="noopener">here.</a>
<h3><strong>Setup Universal Windows Platform app</strong></h3>
UWP application consists of map which displays drivers positions and information label about the connection status with Azure SignalR Hub.

<img class=" wp-image-627 aligncenter" src="/images/devisland/article7/assets/signalrservice10.png?w=300" alt="" width="492" height="277" />

Before you start with SignalR functionality please generate Bing Maps key <a href="https://www.bingmapsportal.com/" target="_blank" rel="noopener">here.</a>

Add "Microsoft.AspNetCore.SignalR.Client" NuGet. Please note that at the moment of writing this article this package is in pre-release version:
<p style="text-align:center;"><img class="alignnone wp-image-655" src="/images/devisland/article7/assets/signalrservice13.png?w=300" alt="" width="482" height="143" /></p>
The same classes called "ClientSignalR" and "LocationUpdate" will be used in this project so please copy its content from Console app described above and place them in "Communication" solution folder.

Now create "Utils" folder and create "MapManager" class. It will be responsible for map setup and displaying push pins with drivers location:

```
    public class MapManager
    {
        private MapControl _map;
        public Dictionary<string, MapIcon> LocationUpdatesDictionary { get; }

        public MapManager(MapControl map)
        {
            _map = map;
            LocationUpdatesDictionary = new Dictionary<string, MapIcon>();
        }

        /// <summary>
        /// Initialize Bing Map.
        /// </summary>
        public void InitializeMap()
        {
            BasicGeoposition centerPosition = new BasicGeoposition { Latitude = 52.2326, Longitude = 20.7810 };
            Geopoint centerPoint = new Geopoint(centerPosition);

            _map.ZoomLevel = 12;
            _map.Center = centerPoint;
        }

        /// <summary>
        /// Add map push pin with location from location update received from SignalR Service.
        /// </summary>
        /// <param name="locationUpdate"></param>
        public void AddMapPushPin(LocationUpdate locationUpdate)
        {
            var landMarks = new List<MapElement>();
            BasicGeoposition snPosition = new BasicGeoposition { Latitude = locationUpdate.Latitude, Longitude = locationUpdate.Longitude };
            Geopoint snPoint = new Geopoint(snPosition);

            var locationMapIcon = new MapIcon
            {
                Location = snPoint,
                NormalizedAnchorPoint = new Point(0.5, 1.0),
                ZIndex = 0,
                Title = locationUpdate.DriverName
            };

            if (locationUpdate.DriverName.Equals("Adam"))
                locationMapIcon.Image = RandomAccessStreamReference.CreateFromUri(new Uri("ms-appx:///Assets/CarIcon1.png"));
            else
                locationMapIcon.Image = RandomAccessStreamReference.CreateFromUri(new Uri("ms-appx:///Assets/CarIcon2.png"));

            landMarks.Add(locationMapIcon);

            var LandmarksLayer = new MapElementsLayer
            {
                ZIndex = 1,
                MapElements = landMarks
            };

            _map.Layers.Add(LandmarksLayer);

            _map.Center = snPoint;
            _map.ZoomLevel = 12;

            LocationUpdatesDictionary.Add(locationUpdate.DriverName, locationMapIcon);
        }

        /// <summary>
        /// Update existing map push pin location.
        /// </summary>
        /// <param name="locationUpdate"></param>
        public void UpdatePushPin(LocationUpdate locationUpdate)
        {
            MapIcon mapIcon;
            LocationUpdatesDictionary.TryGetValue(locationUpdate.DriverName, out mapIcon);
            if (mapIcon != null)
            {
                BasicGeoposition snPosition = new BasicGeoposition { Latitude = locationUpdate.Latitude, Longitude = locationUpdate.Longitude };
                Geopoint snPoint = new Geopoint(snPosition);
                mapIcon.Location = snPoint;
            }
        }
    }
```

Update "MainPage.xaml.cs" code. Once page is opened hub client connects with "TransportHub" to receive location updates:

```
 public sealed partial class MainPage : Page
    {
        private ClientSignalR _hubClient;
        private MapManager _mapHelper;

        public MainPage()
        {
            this.InitializeComponent();
            _mapHelper = new MapManager(TransportMap);
        }

        protected async override void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);
            _mapHelper.InitializeMap();
            await InitializeSignalRClient();
        }

        /// <summary>
        /// Initialize connection with TransportHub and subscribe to receive location updates.
        /// </summary>
        /// <returns></returns>
        private async Task InitializeSignalRClient()
        {
            try
            {
                _hubClient = new ClientSignalR();
                await _hubClient.Initialize("http://localhost:63369/transport");
                _hubClient.SubscribeHubMethod("broadcastMessage");
                _hubClient.OnMessageReceived += (locationUpdate) =>
                {
                    UpdateLocationOnMap(locationUpdate);
                };
            }
            catch (Exception ex)
            {
                InformationLabel.Text = "SignalR Connection error: " + ex.Message;
            }
        }

        private async void UpdateLocationOnMap(LocationUpdate locationUpdate)
        {
            await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                if (!_mapHelper.LocationUpdatesDictionary.Keys.Any(driverName => driverName.Equals(locationUpdate.DriverName, StringComparison.OrdinalIgnoreCase)))
                    _mapHelper.AddMapPushPin(locationUpdate);
                else
                    _mapHelper.UpdatePushPin(locationUpdate);
            });
        }
    }
```

"MainPage.xaml" file should contain map control and connection status label. Please review full code on my GitHub <a href="https://github.com/Daniel-Krzyczkowski/UniversalWindowsPlatform/tree/master/AzureSignalRTransportApp" target="_blank" rel="noopener">here.</a>
<h3><strong>Test solution</strong></h3>
You can test solution locally or publish ASP .NET Core Web API on Azure as Web App. To test on localhost launch applications in such order:
<ul>
 	<li>Web API</li>
 	<li>UWP application</li>
 	<li>Console application</li>
</ul>
Final result should look like below:

<img class=" wp-image-641 aligncenter" src="/images/devisland/article7/assets/signalrservice11.png?w=300" alt="" width="627" height="295" />
<h3><strong>Wrapping up</strong></h3>
In this article I presented how to configure Azure SignalR Service and pass real-time location data to UWP application. You can test this PoC yourself. You can find source code on my <a href="https://github.com/Daniel-Krzyczkowski" target="_blank" rel="noopener">GitHub</a>. I also recommend to see other use cases of SignalR Service on the <a href="https://azure.microsoft.com/en-us/services/signalr-service/" target="_blank" rel="noopener">official website.</a>