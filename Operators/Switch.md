# Switch

> [Switch](http://reactivex.io/documentation/operators/switch.html): convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently-emitted of those Observables

![switch marble diagram](http://reactivex.io/documentation/operators/images/switch.c.png)

## The Problem

You are writing an application that displays the weather in your current location.

Conveniently, your back end has a method that takes in a zip code and yields a stream of temperature updates:

```
Observable<Temperature> temperatures = 
    mWeatherService.temperatureStreamForZipCode("30312")
```

Every time the temperature in the 30312 zip code changes, `temperatures` will yield a new value.

Another part of your back end tells you the current zip code:

```
Observable<String> zipCodes =
    mLocationTracker.currentZipCode();
```

Every time your zip code changes, `zipCodes` will yield a new value.

So you need to take your `temperatures` stream and your `zipCodes` stream and put them together into a single stream of `Temperature` objects: the current temperature, at the current location.

## The Solution

You can use `map` to get a stream of temperatures for each zipCode:

```
Observable<Observable<Temperature>> temperatureStreams = mLocationTracker
    .currentZipCode()
    .map(zipCode -> mWeatherService.temperatureStreamForZipCode(zipCode));
```

Then you can use `switchOnNext` to convert that to a single `Observable<Temperature>` stream.

```
Observable<Observable<Temperature>> temperatureStreams = mLocationTracker
    .currentZipCode()
    .map(zipCode -> mWeatherService.temperatureStreamForZipCode(zipCode));

Observable<Temperature> myTemperature = Observable.switchOnNext(temperatureStreams);
```

`zipCodes` will start out by emitting the current location, "30312," which will then map to a stream of temperatures for "30312."

When you change locations, `zipCodes` will emit the new location ("30307"), which will call `temperatureStreamForZipCode` to get the temperatures for "30307". 
`myTemperature` will then *switch* to using the temperatures for "30307," not "30312."

`switchMap` can be used to accomplish the same thing in a more convenient way:

```
Observable<Temperature> myTemperature = mLocationTracker
    .currentZipCode()
    .switchMap(zipCode -> mWeatherService.temperatureStreamForZipCode(zipCode));
```

`switchMap` is equivalent to calling `map`, then wrapping the result in a call to `switchOnNext`.
