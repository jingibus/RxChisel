# GroupBy

![GroupBy marble diagram](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/groupBy.png)

## The Problem

Your application displays a map of real-estate data driven by a web service:

```
public interface RealEstateService {
    @GET("/region/markers")
    Observable<Marker> getMarkers(@Query("region_id") int regionId);
}
```

`Marker`s may represent houses, condos, townhomes, or unbuilt land:

```
public enum MarkerType {
    HOUSE, CONDO, TOWNHOME, PARCEL
}
```

You already display the total number of markers shown in one area of your interface:

```
Observable<String> totalCount = mRealEstateService.getMarkers(region.getId())
    .scan(0, (sum, marker) -> sum + 1)
    .map(Object::toString);
```

Another part of your screen will show this number broken out into a count of all the houses, all the houses, all the townhomes, and all the parcels.

## The Solution

The `groupBy` operator turns one `Observable` into multiple `Observable`s, assorted by a key value.

Unlike operators like `flatMap`, which are intended to flatten out a nested structure, `groupBy` *adds* nested structure to a flat `Observable`:

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

Observable<GroupedObservable<MarkerType,Marker>> markersByType =
    markers.groupBy(marker -> marker.getType());
```

This short version of `groupBy` takes in one function to provide the key type.
(The longer version allows you to transform each element.)
You can then subscribe to the outer `Observable` and dispatch work based on the key value.

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

Observable<GroupedObservable<MarkerType,Marker>> markersByType =
    markers.groupBy(marker -> marker.getType());

markersByType.subscribe(markerGroup -> {
    MarkerType markerType = markerGroup.getType();
    final TextView textView = getTextView(markerType);

    markerGroup
        .scan(0, (sum, marker) -> sum + 1)
        .subscribe(count -> textView.setText(count.toString()));
});
```

The long version of `groupBy` allows you to transform the emitted elements at the same time.
A display of the total value of property for each type would look like this:

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

Observable<GroupedObservable<MarkerType,Marker>> markersByType =
    markers.groupBy(marker -> marker.getType(), Marker::getPrice);

markersByType.subscribe(markerGroup -> {
    MarkerType markerType = markerGroup.getType();
    final TextView textView = getTextView(markerType);

    markerGroup
        .scan(0.0, (total, price) -> total + price)
        .subscribe(price -> textView.setText(price.toString()));
});
```
