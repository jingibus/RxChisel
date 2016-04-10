# Scan

![Scan marble diagram](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/scanSeed.png)

## The Problem

You are building an application that shows houses for sale.
A web service provides a marker for each house of interest on the map.

```
public interface RealEstateService {
    @GET("/region/markers")
    Observable<Marker> getMarkers(@Query("region_id") int regionId);
}
```

As part of this application, you would like to show a count of the total number of markers in a given region.

A normal `map` or `flatMap` cannot solve this problem, because they operate on a per-item basis, not on multiple items.

## The Solution

The `scan` operator can solve this problem.
It is used the same way as "fold" or "reduce" is used in most functional languages.

Starting with an `Observable<Marker>`:

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());
```

You can use `scan` to accumulate a total count of the number of markers.

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

Observable<Integer> count = markers
    .scan(0, (sum, marker) -> sum + 1);
```

The first parameter to `scan` is the initial value that you will be summing up.
The count here is 0 before anything is counted, so the integer value 0 is the first parameter.

After that, you define a function with two parameters that returns a new value for the sum.
The first parameter is the old sum value, the right parameter is the next item emitted by the `Observable`, and the function returns the old sum value plus 1.

So the first call would be:

```
(0, marker) -> 0 + 1
```

The second would be:

```
(1, marker) -> 1 + 1
```

...and so on.

Here's another example.
You can calculate the total sale price of all the real estate being sold in the area:

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

Observable<Double> totalValue = markers
    .scan(0.0, (total, marker) -> total + marker.getPrice());
```

We can walk through this with houses valued at $100k and $150k.
Again, the evaluation would start at 0.0:

```
(0.0, marker1) -> 0.0 + marker1.getPrice()
=> 0.0 + 100000.0
```

And then the next call would be:

```
(100000.0, marker2) -> 100000.0 + marker2.getPrice()
=> 100000.0 + 150000.0
```

So the final observable would emit:

```
Observable<Marker> markers = mRealEstateService.getMarkers(region.getId());

markers
    .scan(0.0, (total, marker) -> total + marker.getPrice())
    .subscribe((Double totalValue) -> {
        Log.i(TAG, "Total value: " + totalValue);
    });

====>

Total value: 100000.0
Total value: 250000.0
```
