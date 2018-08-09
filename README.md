# Android-Location-Service-fusedlocation-API-
The fused location provider is a location API in Google Play services that intelligently combines different signals to provide the location information that your app needs.

Place this in Manifest XML

<service
            android:name=".MyLocationService"
            android:screenOrientation="portrait" />

Create a java class MyLocationService


public class MyLocationService extends Service {

    FusedLocationProviderClient fusedLocationProviderClient;
    private static final int MY_PERMISSION_REQUEST_FINE_LOCATION = 101;

    private LocationRequest locationRequest;
    private LocationCallback locationCallback;
    private boolean updatesOn = false;

    private TextView latitude;
    private TextView longitude;
    String add;
    Intent i;
    // private TextView altitude;
    // private TextView accuracy;
    // private TextView speed;

    private void call_fused() {

        locationRequest = new LocationRequest();
        locationRequest.setInterval(15000); //use a value fo about 10 to 15s for a real app
        locationRequest.setFastestInterval(4000);
        locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);


        fusedLocationProviderClient = LocationServices.getFusedLocationProviderClient(this);
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {

            return;
        }

        fusedLocationProviderClient.getLastLocation().addOnSuccessListener(new OnSuccessListener<Location>() {
            @Override
            public void onSuccess(Location location) {

                if (location != null) {

                    startLocationUpdates();

                }
            }
        });

        locationCallback = new LocationCallback(){
            @Override
            public void onLocationResult(LocationResult locationResult) {
                super.onLocationResult(locationResult);

                for (Location location : locationResult.getLocations()) {

                    //Update UI with location data
                    if (location != null) {

                        i = new Intent("location_update");
                        i.putExtra("coordinates",location.getLongitude()+" , "+location.getLatitude());


                        //Toast.makeText(getApplicationContext(),location.getLongitude()+" , "+location.getLatitude() , Toast.LENGTH_LONG).show();
                        getAddress(location.getLongitude() , location.getLatitude());

                    }
                }
            }
        };
    }


    @Override
    public void onCreate() {
        super.onCreate();
        call_fused();


    }

    private void startLocationUpdates() {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            fusedLocationProviderClient.requestLocationUpdates(locationRequest, locationCallback, null);
        }
    }

    private void stopLocationUpdates() {
        fusedLocationProviderClient.removeLocationUpdates(locationCallback);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        stopLocationUpdates();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        //Toast.makeText(getApplicationContext() , "FusedProviderClient Started", Toast.LENGTH_LONG).show();

        super.onStartCommand(intent, flags, startId);
        return START_STICKY;

    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }


    public String getAddress(double lng, double lat) {
        Geocoder geocoder = new Geocoder(getApplicationContext() , Locale.getDefault());

        try {
            List<Address> addresses = geocoder.getFromLocation(lat, lng, 1);
            Address obj = addresses.get(0);
            //add = obj.getAddressLine(0);
            add = obj.getSubLocality();
            i.putExtra("area", add);
            sendBroadcast(i);
            // Log.v("Message", "" + add);

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            Toast.makeText(this, e.getMessage(), Toast.LENGTH_SHORT).show();
        }
        return add;

    }

}

