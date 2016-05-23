# FlatMap

![FlatMap marble diagram](http://reactivex.io/documentation/operators/images/flatMap.c.png)

## The Problem

You are building an application that shows houses for sale on a map.
Your web service provides this information through two endpoints:

```
public interface RealEstateService {
    @GET("/regions")
    Observable<Region> getRegions(
        @Query("lon_w") double lon_w, 
        @Query("lat_n") double lat_n,
        @Query("lon_e") double lon_e,
        @Query("lat_s") double lat_s);

    @GET("/region/markers")
    Observable<Marker> getMarkers(@Query("region_id") int regionId);
}
```

You first have to call `getRegions` to ask the web service for a set of regions that your screen is displaying.

Once you have regions, you can then call `getMarkers` for each region to get the markers that should be displayed to show houses for sale, or other items of interest.

While this is a lot of web service calls, you don't have time to wait for a better `getMarkers` call before you ship.
And when you try to use `map`, you get this:

```
Observable<Region> regions = mRealEstateService
    .getRegions(lonW, latN, lonE, latS);

Observable<Observable<Marker>> markers = regions
    .map(region -> mRealEstateService.getMarkers(region.getId()));
```

That doesn't look like what you want, and when you try to use it, it doesn't work.
Each item in `markers` is *itself* an observable, which means you must somehow subscribe to each item:

```
Observable<Region> regions = mRealEstateService
    .getRegions(lonW, latN, lonE, latS);

Observable<Observable<Marker>> markers = regions
    .map(region -> mRealEstateService.getMarkers(region.getId()))
    .map(markers -> markers.subscribe(marker -> plotMarker(marker)));
```

The idea of keeping track of all those subscriptions gives you a headache.

## The Solution

Use `flatMap` instead.

`flatMap` does two things in one operator.
First, it uses the function you pass in to perform a `map`.

```
Observable<Region> regions = mRealEstateService
    .getRegions(lonW, latN, lonE, latS);

Observable<Observable<Marker>> markerObservables = regions
    .map(region -> mRealEstateService.getMarkers(region.getId()));
```

Then it `merge`s those observables together into a single `Observable`.

```
Observable<Region> regions = mRealEstateService
    .getRegions(lonW, latN, lonE, latS);

Observable<Observable<Marker>> markerObservables = regions
    .map(region -> mRealEstateService.getMarkers(region.getId()));

Observable<Marker> markers = Observable.merge(markerObservables);
```

It's as if you `subscribe`d to each one of the `Observable`s returned by `map` at the same time, and then merged the `Marker`s they returned into a single `Observable` stream.

And that's what `flatMap` does.

```
Observable<Region> regions = mRealEstateService
    .getRegions(lonW, latN, lonE, latS);

Observable<Marker> markers = regions
    .flatMap(region -> mRealEstateService.getMarkers(region.getId()));
```

### Ordering

When `getRegions` finishes performing its web service call, it will emit the regions it returns in a specific order.

```
mRealEstateService
    .getRegions(lonW, latN, lonE, latS);
    .flatMap(region -> {
        Log.i(TAG, "Fetching markers for region: " + region.getName());
        return mRealEstateService.getMarkers(region.getId())
    })
    .subscribe();

====> 

Fetching markers for region: Cabbagetown
Fetching markers for region: Inman Park
Fetching markers for region: Little 5 Points
```

These regions be emitted quickly, or they may be called slowly.

Each call to `getMarkers` may not take the same amount of time, either.
So the responses may not arrive in the same order as the calls were made.

`flatMap` is very simple: if the responses arrive out of order, it will emit the values out of order.

```
mRealEstateService
    .getRegions(lonW, latN, lonE, latS);
    .flatMap(region -> {
        Log.i(TAG, "Fetching markers for region: " + region.getName());
        return mRealEstateService.getMarkers(region.getId())
    })
    .subscribe(marker -> {
        Log.i(TAG, "Returned a marker located in: " + marker.getRegion().getName());
    });

====>

Fetching markers for region: Cabbagetown
Fetching markers for region: Inman Park
Fetching markers for region: Little 5 Points
Returned a marker located in: Little 5 Points
Returned a marker located in: Cabbagetown
Returned a marker located in: Little 5 Points
Returned a marker located in: Cabbagetown
Returned a marker located in: Inman Park
Returned a marker located in: Inman Park
Returned a marker located in: Little 5 Points
Returned a marker located in: Inman Park
```

If you need your values to arrive in the same order as they were requested, use `concatMap` instead.

## Related Tools

`concatMap` is like `flatMap`, but it will *not* mix your `Marker`s together or change their ordering.
In this case, that's fine — the user doesn't care about ordering, they just need to see each `Marker` as it can be displayed.
