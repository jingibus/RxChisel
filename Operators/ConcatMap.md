# ConcatMap

![ConcatMap marble diagram](http://reactivex.io/documentation/operators/images/concatMap.png)

## The Problem

You are building an application that shows houses for sale.
You would like to display extended information about houses in a specified region in a list on the screen.

Your web service provides this information through two endpoints:

```
public interface RealEstateService {
    @GET("/region/markers")
    Observable<Marker> getMarkers(@Query("region_id") int regionId);

    @GET("/markers/{marker_id}")
    Observable<MarkerDetails> getMarkerDetails(@Path("marker_id") int markerId);
}
```

To display the information from the `MarkerDetails` objects, `getMarkerDetails` must be called for each `Marker` returned by `getMarkers`.

Doing this with `map` will make the correct web service calls:

```
Region region = ...;

Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());
Observable<Observable<MarkerDetails>> markerDetails = markers
    .map((Marker marker) -> {
        return mRealEstateService.getMarkerDetails(marker.getId());
    });
```

...but it will not wait for those calls to return.

`flatMap` will wait for those calls to return:

```
Region region = ...;

Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());
markers
    .flatMap((Marker marker) -> {
        Log.i(TAG, "Getting details: " + marker.getAddress());
        return mRealEstateService.getMarkerDetails(marker.getId());
    })
    .subscribe((MarkerDetails markerDetails) -> {
        Log.i(TAG, "Received details for: " + markerDetails.getAddress());
    });

====>

Getting details: 123 Boulevard SE
Getting details: 500 Memorial Dr
Getting details: 1000 Moreland Ave
Received details for: 1000 Moreland Ave
Received details for: 500 Memorial Dr
Received details for: 123 Boulevard SE
```

...but it will not preserve the order of the source `Observable`.

Since this list is displayed to the user, it should have a consistent order, not the random order `flatMap` yields.

## The Solution

`concatMap` performs the same underlying operation as `flatMap`, but it preserves the ordering of the source `Observable`.

```
Region region = ...;

Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());
markers
    .concatMap((Marker marker) -> {
        Log.i(TAG, "Getting details: " + marker.getAddress());
        return mRealEstateService.getMarkerDetails(marker.getId());
    })
    .subscribe((MarkerDetails markerDetails) -> {
        Log.i(TAG, "Received details for: " + markerDetails.getAddress());
    });

====>

Getting details: 123 Boulevard SE
Getting details: 500 Memorial Dr
Getting details: 1000 Moreland Ave
Received details for: 123 Boulevard SE
Received details for: 500 Memorial Dr
Received details for: 1000 Moreland Ave
```

## Related Tools

If you do not care about ordering, `flatMap` will emit its results quicker than `concatMap` will.
