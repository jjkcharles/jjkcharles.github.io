---
layout: post
title: RestCountries.Net
subtitle: Offline access to all the countries
author: jjk_charles
categories: explorations
tags: dotnet explorations
---

Accessing accurate and up-to-date information about countries is essential for various applications. [RestCountries RESTful API](https://restcountries.com/) offers a pretty good way to access the same, so what else can be better than that right? There actually is - what if you could access them all offline, in a [type-safe](https://en.wikipedia.org/wiki/Type_safety) way from within your .Net code!

This is exactly what *RESTCountries.Net* offers. The project is referred to by its [creator - @egbakou](https://github.com/egbakou) in the project's [Github page](https://github.com/egbakou/RESTCountries.NET) as below,

```
A completely offline library to get information about countries.
```

You'll have to note that, its not just a wrapper on top of RESTCountries API. So, when you use it to pull details about a country there is no network calls involved.

If you look into the code, you could see that all the information is cached into a JSON [data.json](https://github.com/egbakou/RESTCountries.NET/blob/main/src/RESTCountries.NET/Services) which is where you'll be served the details from.

The library is available as a [nuget package](https://www.nuget.org/packages/RESTCountries.NET/), so installation is as simple as doing the below from your dotnet CLI, 
```bash
> dotnet add package RESTCountries.NET
```

[@egbakou](https://github.com/egbakou) has pretty good documentation both on [project's readme page](https://github.com/egbakou/RESTCountries.NET/blob/main/README.md) as well as in his [website](https://lioncoding.com/getting-started-with-restcountries.net/), so I will not to go into any details about the usage or capabilities. But, below are few things I tried while I was playing around with the library which gives you an idea of what is possible.

### Countries using a specific country
```csharp
var curreny = "INR";
Console.WriteLine("\nCountries that use {0} as currency:", curreny);
foreach(var x in RestCountriesService.GetCountriesByCurrency(curreny))
{
    Console.WriteLine("{0} (Capital: {1})", x.Name.Common, x.Capital.First());
}
```
Below is the output,
```
Countries that use INR as currency:
Bhutan (Capital: Thimphu)
India (Capital: New Delhi)
```

### Translating a country name
```csharp
var code = "ESP";
Console.WriteLine("\nCountry with code {0}:", code);
Console.WriteLine("{0} (Capital: {1})", RestCountriesService.GetCountryByCode(code)?.Name.Common, RestCountriesService.GetCountryByCode(code)?.Capital.First());
Console.WriteLine("In French:");
Console.WriteLine("{0} (Capital: {1})", RestCountriesService.GetCountryByCode(code)?.Translations["fra"].Common, RestCountriesService.GetCountryByCode(code)?.Capital.First());
```
Below is the output,
```
Country with code ESP:
Spain (Capital: Madrid)
In French:
Espagne (Capital: Madrid)
```
I noticed that countries capital cannot be tranlated similar to how we can do the country name itself.

### Countries that share land borders with specified country
```csharp
var inputCountryCode="THA";
var inputCountry = RestCountriesService.GetCountryByCode(inputCountryCode);
if(inputCountry != null)
{
    var borderingCountries = RestCountriesService.GetAllCountries().Where(x=>x.Borders!=null && x.Borders.Contains(inputCountryCode));
    if(borderingCountries.Count()>0) 
    {
        Console.WriteLine("\n{0} shares borders with following countries:", inputCountry?.Name.Common);
    }
    else
    {
        Console.WriteLine("\n{0} shares borders with no other countries.", inputCountry?.Name.Common);
    }
    foreach(var borderCountry in borderingCountries)
    {
        Console.WriteLine("{0} (Capital: {1})", borderCountry.Name.Common, borderCountry.Capital.First());
    }
}
```
Below is the output,
```
Thailand shares borders with following countries:
Cambodia (Capital: Phnom Penh)
Laos (Capital: Vientiane)
Malaysia (Capital: Kuala Lumpur)
Myanmar (Capital: Naypyidaw)
```

You can do much more advanced filtering with all the data available across countries. If you haven't come across this library before or haven't used it thus far, do give it a try.

Happy Coding!