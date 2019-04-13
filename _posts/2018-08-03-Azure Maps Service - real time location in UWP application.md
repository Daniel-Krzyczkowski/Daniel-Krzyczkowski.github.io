---
title: "Azure Maps Service - real time location in UWP application"
---

<p align="center">
<img src="/images/devisland/article10/assets/azuremaps1.png?raw=true" alt="Azure Maps Service - real time location in UWP application"/>
</p>

<h3><strong>Short introduction</strong></h3>
New great service appeared with status "General Availability" in Azure portal - <a href="https://azure.microsoft.com/pl-pl/services/azure-maps/" target="_blank" rel="noopener">Azure Maps</a> . It is a collection of geo-spatial services, backed by fresh mapping data. It contains REST APIs for rendering maps, searching points of interest, routes to points of interests, traffic conditions, time zones, and IP to location services. In this article I would like to present how to use Azure Maps service together with Azure Signal R service to display real time position together with route drawn on the map. I already explained what Azure Signal R service is in my previous article <a href="/images/devisland/article10/assets/real-time-data-with-microsoft-azure-signalr-service/" target="_blank" rel="noopener">here.</a>

In this article I want to show how to display real time location together with route in the UWP application. Please read above article first before moving forward.
<h3><strong>Solution structure</strong></h3>
To simulate real-time transport data I prepared three applications for this solution:
<ul>
 	<li>ASP .NET Core Web API - web API integrated with Azure SignalR service and Azure Maps service to send real-time transport data to client app and return whole route which should be drawn on the map</li>
 	<li>Console App - created to send fake location data to Web API about transport</li>
 	<li>Universal Windows App - client application to display information on the map about driver and the route</li>
</ul>
<h3><strong>Setup Azure Maps Service</strong></h3>
Open <a href="http://portal.azure.com" target="_blank" rel="noopener">Microsoft Azure</a> portal. Type "Azure Maps" in search window:

<img class=" wp-image-1169 aligncenter" src="https://devislandblog.files.wordpress.com/2018/08/azuremaps2.png?w=300" alt="" width="453" height="151" />

Fill required information like name of the service (used later with full URL):

<img class=" wp-image-1171 aligncenter" src="https://devislandblog.files.wordpress.com/2018/08/azuremaps3.png?w=300" alt="" width="435" height="374" />

Once service is created open the tab and select "Keys" section:

<img class=" wp-image-1173 aligncenter" src="https://devislandblog.files.wordpress.com/2018/08/azuremaps4.png?w=300" alt="" width="438" height="334" />

Copy Primary Key because we will use it later in Web API application setup.
<h3><strong>Setup ASP .NET Core Web API</strong></h3>
We will use ASP .NET Core Web API application created in the previous article available <a href="/images/devisland/article10/assets/real-time-data-with-microsoft-azure-signalr-service/" target="_blank" rel="noopener">here.</a> Connection with Signal R Service is already configured but we have to add functionality connected with generating route basing on the coordinates. Here Azure Maps enters the game.

We will comunicate with Azure Maps REST API using RestSharp. Clone project from my Github <a href="https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/AzureSignalRTransportApp" target="_blank" rel="noopener">here</a> , open it. You should see that RestSharp NuGet package is added:

<img class=" wp-image-1184 aligncenter" src="https://devislandblog.files.wordpress.com/2018/08/azuremaps5.png?w=300" alt="" width="695" height="153" />

Now open "MapService" class from "Services" folder:

```
public interface IMapService
    {
        Task<DirectionsResponse> GetDirections(DirectionsRequest directionsRequest);
    }

    public class MapService : IMapService
    {
        private AzureMapsSettings _azureMapsSettings;
        private RestClient _restClient;

        public MapService(IOptions<AzureMapsSettings> azureMapSettings)
        {
            _azureMapsSettings = azureMapSettings.Value;
            _restClient = new RestClient("https://atlas.microsoft.com");
        }

        public async Task<DirectionsResponse> GetDirections(DirectionsRequest directionsRequest)
        {
            var request = new RestRequest("route/directions/json", Method.GET);
            request.AddQueryParameter("subscription-key", _azureMapsSettings.SubscriptionKey);
            request.AddQueryParameter("api-version", "1.0");
            request.AddQueryParameter("query", $"{directionsRequest.FromLatitude},{directionsRequest.FromLongitude}:{directionsRequest.ToLatitude},{directionsRequest.ToLongitude}");
            var response = await _restClient.ExecuteTaskAsync<string>(request, default(CancellationToken));
            var directions = JsonConvert.DeserializeObject<DirectionsResponse>(response.Data);
            return directions;
        }
    }
```

As you can see we have to pass from and to coordinates (latitude and longitude). We have to also provide map key copied from Azure portal - it is passed in "subscription-key" header. In this case we are using directions endpoint to retrieve route between two points on the map.

In the "Model" folder you can find two classed related with Azure Maps API request: "DirectionsRequest" - instance of this class is passed to "GetDirections" method.  There is also "DirectionsResponse" - this class represents data returned form the Azure Maps.

When you open MapsController class you should see Post method with "DirectionsRequst" parameter:

```
 public class MapController : Controller
    {
        private IMapService _mapService;

        public MapController(IMapService mapService)
        {
            _mapService = mapService;
        }

        [HttpPost]
        public async Task<IActionResult> Post([FromBody] DirectionsRequest directionsRequest)
        {
            var directions = await _mapService.GetDirections(directionsRequest);
            if (directions == null)
                return NotFound();
            return Ok(directions);
        }
    }
```

<h3><strong>Setup .NET Core Console App</strong></h3>
I modified the code of test console application. We are using it to simulate position change of the driver - then we are displaying new position on the map in the UWP application. You can find updated code <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/TransportAppSimulator" target="_blank" rel="noopener">here.</a>

```
    class Program
    {
        static void Main(string[] args)
        {
            List<LocationUpdate> locationUpdates = new List<LocationUpdate>
            {
             new LocationUpdate
                {
                    Latitude = 52.23292,
                    Longitude = 20.99114,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 52.25229,
                    Longitude = 20.94341,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 52.19462,
                    Longitude = 20.82109,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 52.09011,
                    Longitude = 20.36584,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 52.01813,
                    Longitude = 19.97684,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 51.87744,
                    Longitude = 19.64773,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 51.13844,
                    Longitude = 16.93472,
                    DriverName = "Daniel"
                },
             new LocationUpdate
                {
                    Latitude = 51.12705,
                    Longitude = 16.92196,
                    DriverName = "Daniel"
                },
            };

            var hubClient = new ClientSignalR();
            hubClient.Initialize("http://localhost:63369/transport");

            Observable
            .Interval(TimeSpan.FromSeconds(3))
            .Subscribe(
                async x =>
                {
                    var locationUpdate = locationUpdates.FirstOrDefault();
                    if(locationUpdate !=null)
                    {
                        await hubClient.SendHubMessage("broadcastMessage", locationUpdate);
                        Console.WriteLine("SENDING LOCATION UPDATE: " + locationUpdate.DriverName + " " + locationUpdate.Latitude + " " + locationUpdate.Longitude);
                        locationUpdates.Remove(locationUpdate);
                    }
                    else
                    Console.WriteLine("UPDATES COMPLETED");
                });

            Console.ReadKey();
        }
    }
```

<h3><strong>Setup Universal Windows Platform app</strong></h3>
I used previously created UWP application and added functionality related with displaying route on the map. Now application displays route using Azure Maps and live location updates using Signal R Service. You can see updated source code <a href="https://github.com/Daniel-Krzyczkowski/UniversalWindowsPlatform/tree/master/AzureMapsApp" target="_blank" rel="noopener">here.</a>

Open "Services" folder and "MapService" class. As you can see I am using RestSharp to make request to previously created Web API and retrieve directions data to display route on the map.

```
 public class MapService
    {
        private RestClient _restClient;

        public MapService()
        {
            _restClient = new RestClient("http://localhost:63369/api");
        }

        public async Task<DirectionsResponse> GetDirections(DirectionsRequest directionsRequest)
        {
            var request = new RestRequest("map", Method.POST);
            request.AddParameter("application/json; charset=utf-8", JsonConvert.SerializeObject(directionsRequest), ParameterType.RequestBody);
            var response = await _restClient.ExecuteTaskAsync<string>(request, default(CancellationToken));
            var directions = JsonConvert.DeserializeObject<DirectionsResponse>(response.Data);
            return directions;
        }
    }
```

Now open "MapManager" class - here I added "DisplayRoute" method. As a parameter there is response from Azure Maps - "DirectionsResponse". New MapPolyline object is created. Please note that path property is instantiated with response from Azure Maps to display the route. Then new list with map elements is created and at the end we are adding new layer to the map with the route:

```
public void DisplayRoute(DirectionsResponse directions)
        {
            MapPolyline routeLine = new MapPolyline()
            {
                Path = new Geopath(directions.routes[0].legs[0].points.Select(p => new BasicGeoposition { Latitude = p.latitude, Longitude = p.longitude })),
                StrokeColor = Colors.Black,
                StrokeThickness = 3,
                StrokeDashed = true
            };


            var mapLines = new List<MapElement>();

            mapLines.Add(routeLine);

            var LinesLayer = new MapElementsLayer
            {
                ZIndex = 1,
                MapElements = mapLines
            };

            _map.Layers.Add(LinesLayer);
        }
```

<h3><strong>Test solution</strong></h3>
You can test solution locally or publish ASP .NET Core Web API on Azure as Web App. To test on localhost launch applications in such order:
<ul>
 	<li>Web API -</li>
 	<li>UWP application</li>
 	<li>Console application</li>
</ul>
Final result should look like below:

<img class="alignnone wp-image-1157 aligncenter" src="/images/devisland/article10/assets/azuremaps1.png?w=300" alt="" width="529" height="351" />
<h3><strong>Wrapping up</strong></h3>
In this article I presented how to configure Azure Maps service together with Azure SignalR Service and pass real-time location data to UWP application including display of the current route. You can test this PoC yourself. You can find source code on my GitHub - <a href="https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/AzureSignalRTransportApp" target="_blank" rel="noopener">Web API app</a>, <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/TransportAppSimulator" target="_blank" rel="noopener">console test app</a>, <a href="https://github.com/Daniel-Krzyczkowski/UniversalWindowsPlatform/tree/master/AzureMapsApp" target="_blank" rel="noopener">UWP application.</a>