# Argentina daylight

Yesterday one of our client-side teammates, based on the US, was very surprised about Argentina's southern location
and its closeness to Antarctica, after looking it up on Maps out of curiosity.

Quoting him, just for the sake of better representing his surprise 😁, he said:

— "Holy crap was looking at a map"

— "I did **not** realize argentina was that far south"

— "What time is sunset there??"

After thinking a second about Argentina's geography I answered:

— "Well, we have a long latitude range, so sunrise/sunset times vary significantly"

And then couldn't stop myself from wanting to tell him precisely the sunrise and sunset times of northernmost and
southernmost locations of the country.

So I ended up writing a short program which -taking advantage of a couple of external HTTP services for geo-coding and
calculating sunrise and sunset times- prints this output:

```
La Quiaca, Municipio de La Quiaca, Departamento Yavi, Jujuy, 4650, Argentina (-22.1046124,-65.596754):
 - Summer Solstice: 06:36:22 → 20:05:40
 - Winter Solstice: 08:00:57 → 18:47:55
Buenos Aires, Ciudad Autónoma de Buenos Aires, Argentina (-34.6075682,-58.4370894):
 - Summer Solstice: 05:38:00 → 20:06:45
 - Winter Solstice: 08:00:43 → 17:50:52
Ushuaia, Departamento Ushuaia, Tierra del Fuego, Argentina (-54.8069332,-68.3073246):
 - Summer Solstice: 04:51:52 → 22:11:53
 - Winter Solstice: 09:58:57 → 17:11:37
```

Then -myself been trying to share Golang practicality with everyone- I thought "Hey, this short program is a nice
little sample of idiomatic Go and good use of its awesome standard library".

So I decided to share it... and you can read it below or download it from [here](argentina-daylight/main.go).

To run it you just need to execute:

```sh
go run main.go
```

That's -of course- assuming that you have a Go toolkit installed.
If you don't, just follow the instructions at https://golang.org/doc/install, I promise you'll have it installed in
less than a minute.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/url"
	"strconv"
	"time"
)

func main() {
	places := []string{
		"la quiaca, argentina",    // northernmost city
		"buenos aires, argentina", // capital
		"ushuaia, argentina",      // southernmost city
	}
	art, err := time.LoadLocation("America/Buenos_Aires")
	if err != nil {
		log.Fatal(err)
	}
	dates := []struct {
		Name string
		Date time.Time
	}{
		{"Summer Solstice", time.Date(2020, 12, 21, 0, 0, 0, 0, art)},
		{"Winter Solstice", time.Date(2020, 06, 21, 0, 0, 0, 0, art)},
	}
	for _, place := range places {
		locs, err := Locations(place)
		if err != nil {
			fmt.Printf("error locating %q: %v\n", place, err)
			continue
		}
		loc := locs[0]
		fmt.Printf("%s:\n", &loc)
		for _, date := range dates {
			daylight, err := loc.Daylight(date.Date)
			if err != nil {
				fmt.Printf("error getting %s times for %+v: %v\n", date.Name, loc, err)
				continue
			}
			fmt.Printf(" - %s: %s\n", date.Name, daylight)
		}
	}
}

func Locations(query string) ([]Location, error) {
	r, err := http.Get("https://nominatim.openstreetmap.org/search?format=json&q=" + url.QueryEscape(query))
	if err != nil {
		return nil, err
	}
	defer r.Body.Close()
	bs, err := ioutil.ReadAll(r.Body)
	if err != nil {
		return nil, err
	}
	var locs []Location
	err = json.Unmarshal(bs, &locs)
	if err != nil {
		return nil, err
	}
	return locs, nil
}

type Location struct {
	Name string `json:"display_name"`
	Lat  string
	Lon  string
}

func (l *Location) Coordinates() (float64, float64) {
	lat, _ := strconv.ParseFloat(l.Lat, 64)
	lon, _ := strconv.ParseFloat(l.Lon, 64)
	return lat, lon
}

func (l *Location) Daylight(date time.Time) (*Daylight, error) {
	lat, lon := l.Coordinates()
	return LocationDaylight(lat, lon, date)
}

func (l *Location) String() string {
	return fmt.Sprintf("%s (%s,%s)", l.Name, l.Lat, l.Lon)
}

func LocationDaylight(lat, lon float64, date time.Time) (*Daylight, error) {
	r, err := http.Get(fmt.Sprintf("https://api.sunrise-sunset.org/json?formatted=0&lat=%f&lng=%f&date=%s", lat, lon, date.Format("2006-01-02")))
	if err != nil {
		return nil, err
	}
	defer r.Body.Close()
	bs, err := ioutil.ReadAll(r.Body)
	if err != nil {
		return nil, err
	}
	var b struct {
		Results json.RawMessage
		Status  string
	}
	err = json.Unmarshal(bs, &b)
	if err != nil {
		return nil, err
	}
	if b.Status != "OK" {
		return nil, fmt.Errorf("error getting (%f, %f) daylight time: %s", lat, lon, b.Status)
	}
	var sun Daylight
	err = json.Unmarshal(b.Results, &sun)
	if err != nil {
		return nil, err
	}
	sun.Sunrise = sun.Sunrise.In(date.Location())
	sun.Sunset = sun.Sunset.In(date.Location())
	return &sun, nil
}

type Daylight struct {
	Sunrise time.Time
	Sunset  time.Time
}

func (s *Daylight) String() string {
	return fmt.Sprintf("%s → %s", s.Sunrise.Format("15:04:05"), s.Sunset.Format("15:04:05"))
}
```
