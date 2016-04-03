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

When you call `map` and create `markerObservables`, many web service calls may be initiated simultaneously.
When you merge them together, `merge` will emit each `Marker` as it arrives, no matter which `Observable` it came from in `markerObservables`.
So `flatMap` may mix all of your `Marker`s together or change their ordering, depending on when those web service calls return.

## Related Tools

`concatMap` is like `flatMap`, but it will *not* mix your `Marker`s together or change their ordering.
In this case, that's fine — the user doesn't care about ordering, they just need to see each `Marker` as it can be displayed.
