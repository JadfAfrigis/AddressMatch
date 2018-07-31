# e4 Strategic Address Match

## Address Match API Description

The e4 Strategic Address Match web service provides a score for address pairs which indicates the degree of certainty that the addresses in the pair refer to the same location/address. Address pairs are assigned to various categories, by a classification algorithm, depending on the attributes shared by the geocoding results of the addresses in the pair. The address categories, or classes, have associated _ratings_, the score assigned to indicate the degree of certainty that the addresses in the pair refer to the same location/address. The address match web service has intended applications in RICA and FICA.

## This Document

This document serves as a succinct guide for users of the Address Match API, i.e. mostly developers external to AfriGIS, to:
1. demonstrate the function of the Address Match API,
1. explain its input parameters and output properties,
1. provide guidance on API authentication, and
1. exhibit a sample request and response.

## Request arguments

The table below defines the input parameters of the Address Match API.

| Argument | Definition |
|  --- |  --- |
| Input1 | The first address in the address pair. |
| Input2 | The second address in the address pair. |
| include | An opt-in argument to optionally specify which of the following indicator fields *isSectionalScheme*, *isFarm*, *IsHolding* and *isEstate* must be included in the response.  These indicator fields indicate whether at least one of the addresses in the address pair has a geocoding result that is a sectional scheme, a farm, a holding or an estate, respectively. Multiple indicator fields can be included in the response by assigning a comma-separated list to the *include* parameter, e.g. &include=isSectionalScheme,isFarm,isHolding,isEstate. |

### Sample Request

The sample request below depicts the structure of a request to the Address Match API. For more information on AfriGIS SaaS API authentication visit [AfriGIS Developers](https://developers.afrigis.co.za/api-authentication/), alternatively review the JavaScript code sample below which is excerpted from a Postman collection pre-request script.

```
https://saas.afrigis.co.za/rest/2/client.e4.addresscompare/<SaaS client key>/<HMAC SHA-1 authentication code>/?input1=103+JOAN+AVENUE+ELDORADO+PARK++1811&input2=103+JOAN+AVE++1811&include=isSectionalScheme,isEstate,isFarm,isHolding
```

#### Calculating HMAC SHA-1 Authentication Code

The sample JavaScript code below (excerpted from a Postman collection pre-request script for the Address Match API) details how to calculate the HMAC SHA-1 authentication code, which must be included in requests made to AfriGIS SaaS. __Note that the query-string component associated with the opt-in parameters must be included in the string used to calculate HMAC SHA-1 authentication code.__ The code sample includes explanations and variable definitions in comments. Note that Postman environment variables are referenced in this code excerpt.

A Postman collection and a Postman environment are available to demonstrate the Address Match API.

```javascript

//URL encode input1 and input2 - the input addresses to be compared (request parameters).
var i1=encodeURIComponent(pm.variables.get("input1")).replace(/%20/g,'+');
var i2=encodeURIComponent(pm.variables.get("input2")).replace(/%20/g,'+');

//set querystring variable - a portion of the string used to calculate the 
//HMAC SHA-1 authentication code which must be included
//in requests made to AfriGIS SaaS.
var querystring="input1=" + i1 + "&input2=" + i2 + "&include=isSectionalScheme,isEstate,isFarm,isHolding";

//set message variable - one of the components used to calculate the 
//HMAC SHA-1 authentication code which must be included
//in requests made to AfriGIS SaaS.
//client = AfriGIS SaaS client-key
//web_service = client.e4.addresscompare
var message=querystring + "/" + pm.environment.get("web_service") + "/" + pm.environment.get("client");

//calculate authorization code.
//message is an encoded string including the following 
//components: query-string; web-service; AfriGIS SaaS client-key.
var auth= calcHmac(message, pm.environment.get("secret"));
//replace specific characters in the authentication code
auth=auth.replace(/\+/g,'-').replace(/\//g,"_").replace(/=/g,"");

//set authCode variable value - used in a pre-request script in 
//the Postman collection for the Address Match API.
pm.environment.set("authCode", auth);

//set i1 and i2 - encoded-versions of the input addresses to be 
//used in a pre-request script in 
//the Postman collection for the Address Match API.
pm.environment.set("input1e", i1);
pm.environment.set("input2e", i2);

//function to calculate HMAC SHA-1 authentication code.
function calcHmac(hmacMsgToEncode, saasSecretKey) {

    var re1 = new RegExp('(\r\n|\n)', 'g');

    hmacMsgToEncode = hmacMsgToEncode.replace(re1,'\n');

    var hmac = CryptoJS.algo.HMAC.create(CryptoJS.algo.SHA1, saasSecretKey);

    hmac.update(hmacMsgToEncode);

    var hash = hmac.finalize();

    var hashInBase64 = CryptoJS.enc.Base64.stringify(hash);
   
    return hashInBase64;
}
```

## Response properties

The response properties of the Address Match API are detailed in the table below.

| Property/key | Definition |
|  --- |  --- |
| code | Is a 3 digit numeric code, based on HTTP status codes. |
| message | Is an optional message element which is used to indicate if dependencies of the web service were unresponsive for the request. |
| result | Encapsulates the data from the web service. |
| sameAddress | A property to indicate if the geocoded inputs refer to the same address. |
| sameCoordinates | A property to indicate if the geocoded inputs have the same coordinates. |
| qtime | The amount of time in milliseconds from reception of the address match request to the submission of the result. |
| rating | A numeric rating to classify the address pair congruency/similarity. |
| description | A description of the address match result (the criteria used for the rating of the address pair). |
| isSectionalScheme | A boolean field to indicate if one or more of the geocoded inputs is (or is associated with) a sectional scheme. |
| isFarm | A boolean field to indicate if one or more of the geocoded inputs is (or is associated with) a farm. |
| isEstate | A boolean field to indicate if one or more of the geocoded inputs is (or is associated with) an estate. |
| isHolding | A boolean field to indicate if one or more of the geocoded inputs is (or is associated with) a holding. |
| geocodedInput1 | The geocoding result of the first input address. |
| geocodedInput2 | The geocoding result of the second input address. |
| docId | A document identifier denoting a place and or geographic location. Note: The docid is deprecated in favour of the seoid. |
| seoId |  A unique stable textual identifier denoting a place and or geographic location. |
| formatted_address | The formatted address string of the geocoded input. |
| confidence | The [geocoding confidence level](https://git.afrigis.co.za/afrigis-marketing/e4strategic-addressmatch-poc/uploads/caf38cc0334df9a47a5048421b27ae2a/conf.PNG). |
| address_components | The [address components](https://developers.afrigis.co.za/portfolio/search/#address_types) of the geocoded input. |
| type | The [type of the address component](https://developers.afrigis.co.za/portfolio/search/#address_types). |
| administrative_type | The [administrative type of the address component](https://developers.afrigis.co.za/portfolio/search/#address_types). |
| long_name | The full name of the address component. |
| short_name | An abridged or abbreviated name of the address component. |
| geometry | The geometry property (including the latitude and longitude) of the geocoding result. |
| lat | The latitude. |
| lng | The longitude. |
| types | The [address types of the geocoding result](https://developers.afrigis.co.za/portfolio/search/#address_types). |
| source | Is a unique string indicating where the JSON response was created. |

### Sample Response

A sample response from the Address Match API is included below.

```javascript
{
    "code": 200,
    "result": {
        "sameAddress": false,
        "sameCoordinates": false,
        "qtime": 185,
        "rating": null,
        "geocodedInput1": {
            "docId": "AG_STREETS|3~937~13067~36869",
            "seoId": "r1vS4u4rSI_bFKpyqci-LkQ4022d2lkq05",
            "formatted_address": "Jenn Street, Eldorado Park, Soweto, Gauteng, 1811",
            "confidence": 5,
            "address_components": [
                {
                    "type": "route",
                    "administrative_type": "route",
                    "long_name": "Jenn Street",
                    "short_name": "Jenn Street"
                },
                {
                    "type": "sublocality",
                    "administrative_type": "suburb",
                    "long_name": "Eldorado Park",
                    "short_name": "Eldorado Park"
                },
                {
                    "type": "locality",
                    "administrative_type": "town",
                    "long_name": "Soweto",
                    "short_name": "Soweto"
                },
                {
                    "type": "administrative_area_level_2",
                    "administrative_type": "district municipality",
                    "long_name": "City of Johannesburg",
                    "short_name": "JHB"
                },
                {
                    "type": "administrative_area_level_1",
                    "administrative_type": "province",
                    "long_name": "Gauteng",
                    "short_name": "GT"
                },
                {
                    "type": "country",
                    "administrative_type": "country",
                    "long_name": "South Africa",
                    "short_name": "ZA"
                },
                {
                    "type": "postal_code",
                    "administrative_type": "post_box",
                    "long_name": "1813",
                    "short_name": "1813"
                },
                {
                    "type": "postal_code",
                    "administrative_type": "post_street",
                    "long_name": "1811",
                    "short_name": "1811"
                }
            ],
            "geometry": {
                "location": {
                    "lat": -26.300293,
                    "lng": 27.9128571
                }
            },
            "types": [
                "route"
            ]
        },
        "geocodedInput2": {
            "docId": "AG_STREETS|3~140~0~43328",
            "seoId": "xSY8u0B839_oDXwUOzh-wqm405263638",
            "formatted_address": "Joan Avenue, De Deur, Gauteng",
            "confidence": 5,
            "address_components": [
                {
                    "type": "route",
                    "administrative_type": "route",
                    "long_name": "Joan Avenue",
                    "short_name": "Joan Avenue"
                },
                {
                    "type": "locality",
                    "administrative_type": "town",
                    "long_name": "De Deur",
                    "short_name": "De Deur"
                },
                {
                    "type": "administrative_area_level_2",
                    "administrative_type": "district municipality",
                    "long_name": "Sedibeng",
                    "short_name": "DC42"
                },
                {
                    "type": "administrative_area_level_1",
                    "administrative_type": "province",
                    "long_name": "Gauteng",
                    "short_name": "GT"
                },
                {
                    "type": "country",
                    "administrative_type": "country",
                    "long_name": "South Africa",
                    "short_name": "ZA"
                }
            ],
            "geometry": {
                "location": {
                    "lat": -26.45367,
                    "lng": 27.9908218
                }
            },
            "types": [
                "route"
            ]
        },
        "isEstate": false,
        "isFarm": false,
        "isHolding": false,
        "isSectionalScheme": false,
        "description": "Unclassified"
    },
    "source": "e4s.addressmatch.poc.addresscompare"
}
```
