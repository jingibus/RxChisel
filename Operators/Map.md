# Map

![Map marble diagram](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/map.png)

## The Problem

Your application needs to pull temperature data from a web service vending JSON payloads.
A GET request against `https://www.weatherland.com/zipcode/30312` yields the following payload:

```
{ "fahrenheit" : 48.0 }
```

You have a model object that corresponds to this:

```
public class WebTemperature {
    private double mFahrenheit;

    public double getFahrenheit() {
        return mFahrenheit;
    }

    public void setFahrenheit(double fahrenheit) {
        mFahrenheit = fahrenheit;
    }
}
```

You have used a couple of third party libraries to automatically perform the web requests, translates the JSON payload into a `WebTemperature`, and wrap it all up into an interface that returns an `Observable<WebTemperature>`:

```
public interface WeatherlandService {
    @GET("zipcode/{zipcode}")
    Observable<WebTemperature> getTemperature(@Path("zipcode") String zipcode);
}
```


Hooray — a big part of your work is now done.

There are two problems, though:

1. You do not want your front end to consume `WebTemperature`s, in case the web service's API changes in the future.
2. Your front end displays the time the temperature was received, and can also be configured to display the temperature in Celsius. Your front end really wants an instance of this:

```
public class Temperature {
    private long mTimestampMillis;
    private double mFahrenheit;

    public double getFahrenheit() { ... }
    public void setFahrenheit(double fahrenheit) { ... }

    public long getTimestampMillis() { ... }
    public void setTimestampMillis(long timestampMillis) { ... }

    public double getCelsius() {
        return getFahrenheit() - 32 * 5 / 9;
    }
```

You want to translate your `Observable<WebTemperature>` into an `Observable<Temperature>`.

## The Solution

The `map` operator performs exactly this kind of conversion.
It takes in a converter function which it calls on every element.

So, given an `Observable<WebTemperature>`:

```
Observable<WebTemperature> webTemperature = mWeatherlandService
    .getTemperature("30312");
```

You can convert it to an `Observable<Temperature>` like so:

```
Observable<WebTemperature> webTemperature = mWeatherlandService
    .getTemperature("30312");

Observable<Temperature> temperature = webTemperature
    .map(webTemperature -> {
        Temperature temperature = new Temperature();
        temperature.setTimestampMillis(System.currentTimeMillis());
        temperature.setFahrenheit(webTemperature.getFahrenheit());
        return temperature;
    });
```

## Related Tools

* If you are only performing a cast, then you might as well use `cast` instead of `map`.
* If your `map` turns an `Observable` into an `Observable<Observable>`, then `flatMap` and friends are often the right tool.
