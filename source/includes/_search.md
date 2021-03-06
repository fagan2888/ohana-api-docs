# Search Locations

## Get locations that match certain criteria.

```ruby
# Provide an API Endpoint
Ohanakapa.configure do |c|
  c.api_endpoint = 'https://ohana-api-demo.herokuapp.com/api'
end

# Fetch all locations within 2 miles of ZIP code 94103
Ohanakapa.search('search', { location: '94103', radius: 2 })
```

```shell
curl "https://ohana-api-demo.herokuapp.com/api/search?location=94103&radius=2" -H "User-Agent: MyClient/1.0.0"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 22,
    "active": true,
    "admin_emails": [ ],
    "coordinates": [
      -122.4456187,
      37.7517064
    ],
    "description": "Testing adding a new location",
    "latitude": 37.7517064,
    "longitude": -122.4456187,
    "name": "New location test",
    "short_desc": null,
    "slug": "new-location-test",
    "website": null,
    "updated_at": "2014-09-10T08:21:14.853-07:00",
    "contacts_url": "https://ohana-api-demo.herokuapp.com/api/locations/new-location-test/contacts",
    "services_url": "https://ohana-api-demo.herokuapp.com/api/locations/new-location-test/services",
    "url": "https://ohana-api-demo.herokuapp.com/api/locations/new-location-test",
    "address": {
      "id": 28,
      "address_1": "1290 Ridder Park Drive",
      "address_2": null,
      "city": "San Francisco",
      "state_province": "CA",
      "postal_code": "94103"
    },
    "organization": {
      "id": 8,
      "accreditations": [],
      "alternate_name": null,
      "date_incorporated": null,
      "description": "Test description",
      "email": null,
      "funding_sources": [],
      "licenses": [],
      "name": "Admin Test Org",
      "slug": "admin-test-org",
      "website": null,
      "url": "https://ohana-api-demo.herokuapp.com/api/organizations/admin-test-org",
      "locations_url": "https://ohana-api-demo.herokuapp.com/api/organizations/admin-test-org/locations"
    },
    "phones": [
      {
        "id":43,
        "department":"Information",
        "extension":"123",
        "number":"650 372-6200",
        "number_type":"voice",
        "vanity_number":"222-555-INFO"
      },
      {
        "id":44,
        "department":"Reservations",
        "extension":null,
        "number":"650 627-8244",
        "number_type":"fax",
        "vanity_number":null
      }
    ]
  },
  {
    "id": 15,
    "active": true,
    "admin_emails": [ ],
    "coordinates": [
      -122.4136494,
      37.7756408
    ],
    "description": "Location with no service",
    "latitude": 37.7756408,
    "longitude": -122.4136494,
    "name": "Location with no service",
    "short_desc": null,
    "slug": "location-with-no-service",
    "website": null,
    "updated_at": "2014-09-10T08:21:14.848-07:00",
    "contacts_url": "https://ohana-api-demo.herokuapp.com/api/locations/location-with-no-service/contacts",
    "services_url": "https://ohana-api-demo.herokuapp.com/api/locations/location-with-no-service/services",
    "url": "https://ohana-api-demo.herokuapp.com/api/locations/location-with-no-service",
    "address": {
      "id": 27,
      "address_1": "155 9th St",
      "address_2": null,
      "city": "San Francisco",
      "state_province": "CA",
      "postal_code": "94103"
    },
    "organization": {
      "id": 8,
      "accreditations": [],
      "alternate_name": null,
      "date_incorporated": null,
      "description": "Test description",
      "email": null,
      "funding_sources": [],
      "licenses": [],
      "name": "Admin Test Org",
      "slug": "admin-test-org",
      "website": null,
      "url": "https://ohana-api-demo.herokuapp.com/api/organizations/admin-test-org",
      "locations_url": "https://ohana-api-demo.herokuapp.com/api/organizations/admin-test-org/locations"
    },
    "phones": [ ]
  }
]
```

This endpoint retrieves all locations that match the criteria based on the parameters below. Multiple parameters can be used at the same time, separated by an ampersand (`&`).

### HTTP Request

`GET https://ohana-api-demo.herokuapp.com/api/search{?category,email,keyword,language,lat_lng,location,org_name,radius,service_area,status}`

### Sorting of results
Except for location-based and keyword-based searches, results are sorted by location `id` in ascending order. Location-based searches (those that use the `lat_lng` or `location` parameter) are sorted by distance, with the ones closest to the search query appearing first. `keyword` searches are sorted by relevance since they perform a full-text search in various fields across various tables. See the `keyword` parameter description below for more details.

### URL Query Parameters

Parameter | Description
--------- | -----------
category | The name of the Service category. By default, Ohana API uses the [Open Eligibility](http://openeligibility.org/) taxonomy for Human Services. The parameter must be spelled exactly as in the Open Eligibility taxonomy, including capitalizations and spaces. Some deployments might choose to use their own taxonomy. In that case, refer to their documentation. You can either pass in one category as a String or multiple categories as an Array. For example: `https://ohana-api-demo.herokuapp.com/api/search?category[]=Transit&category[]=Care`. This will return Locations with Services that are associated with either the `Transit` or `Care` categories.
email | This is only useful if you're building an external admin interface. It allows you to determine which locations the user that just signed in has access to. When you pass in an email to the API, it first checks to see if the domain name portion of the email matches one of the [generic domains](https://github.com/codeforamerica/ohana-api/blob/master/config/settings.yml#L29-54) defined in settings.yml. If it does, then it finds locations whose `admin_emails` matches the user's email or whose `email` matches the email. Otherwise, if it doesn't match a generic domain, then it will find locations whose `admin_emails` matches the email, or whose `website` matches the domain name, or whose `email` matches the domain name. Read the [source code](https://github.com/codeforamerica/ohana-api/blob/master/app/models/concerns/search.rb#L23-40) for more details.
keyword | You can pass in one or more words, and the API will look for the occurrence of those words in the following fields: the organization's name, the location's name, the location's description, the service's name, the service's description, the service's keywords, and the service's categories. If more than one word is passed, the API will return matches where **all** of the words are present (but not necessarily as a phrase). Matches within the location's description, the service's description, and the service's category names are ranked higher than matches in the other fields. This full text search is performed via a Postgres [tsvector column](https://github.com/codeforamerica/ohana-api/blob/master/db/migrate/20140508194831_update_search_vector_migration.rb) and the [pg_search](https://github.com/Casecommons/pg_search) gem.
language | Returns locations whose `languages` field matches the parameter. You can either pass in one language as a String or multiple languages as an Array. For example: `https://ohana-api-demo.herokuapp.com/api/search?language[]=Spanish&language[]=French`. This will return locations whose `languages` field contains either `Spanish` or `French`.
lat_lng | Find locations near a specific latitude and longitude. This parameter must be a comma-delimited lat,long pair of floats. For example: `https://ohana-api-demo.herokuapp.com/api/search?lat_lng=37.477227,-122.213221`. By default, this will find locations within a 5-mile radius of those coordinates. To use a different radius, pass in the radius parameter (see description below).
location | Find locations near a specific address or ZIP code. For example: `https://ohana-api-demo.herokuapp.com/api/search?location=94403`. By default, this will find locations within a 5-mile radius of those coordinates. To use a different radius, pass in the radius parameter (see description below).
org_name | Find locations that belong to an Organization whose name matches the parameter. If more than one word is passed, the API will return matches where **all** of the words are present (but not necessarily as a phrase).
radius | Use this parameter in conjunction with the `location` or `lat_lng` parameter to define the search area in miles. The default value is 5, the minimum value is 0.1, and the maximum value is 50.
service_area | Find locations whose services are available in a particular service area. For example: `https://ohana-api-demo.herokuapp.com/api/search?service_area=belmont`. The list of valid service areas will usually be defined in a deployment's [settings.yml](https://github.com/codeforamerica/ohana-api/blob/master/config/settings.yml#L152-172).
status | If set to `active`, it finds Locations that have at least one active Service. If set to `inactive`, it finds Locations that don't have any active Services.
page | The particular page of results. Default is 1.
per_page | Amount of locations to return per page, up to 100. Default is 30.
