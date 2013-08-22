## The application

We've created a small application in [Go](http://golang.org) that prints a list of cities on request. The application logic can be found in [`app.go`](https://github.com/pjvds/go-cities-appengine/blob/master/app.go):

``` go
package cities

import (
	"encoding/json"
	"net/http"
)

var (
	// The cities that we will serve
	cities = []string{
		"Amsterdam", "San Francisco", "Paris", "New York", "Portland",
	}
)

func init() {
	// Register the index handler to the
	// default DefaultServeMux.
	http.HandleFunc("/", handleIndex)
}

func handleIndex(rw http.ResponseWriter, req *http.Request) {
	rw.Header().Set("Content-Type", "application/json")

	encoder := json.NewEncoder(rw)
	encoder.Encode(cities)
}
```

In the `init` method we register `handleIndex` to the `http.DefaultServeMux` which is used by Google's AppEngine.

To make sure everything is working as expected we've added a few tests in [`app_test.go`](https://github.com/pjvds/go-cities-appengine/blob/master/app_test.go):

``` go
package cities

import (
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"
)

func TestHandleIndexReturnsWithStatusOK(t *testing.T) {
	request, _ := http.NewRequest("GET", "/", nil)
	response := httptest.NewRecorder()

	http.DefaultServeMux.ServeHTTP(response, request)

	if response.Code != http.StatusOK {
		t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "200", response.Code)
	}
}

func TestHandleIndexContainsAmsterdam(t *testing.T) {
	request, _ := http.NewRequest("GET", "/", nil)
	response := httptest.NewRecorder()

	http.DefaultServeMux.ServeHTTP(response, request)

	body := response.Body.String()
	if !strings.Contains(body, "Amsterdam") {
		t.Fatalf("Response body did not contain expected %v:\n\tbody: %v", "Amsterdam", body)
	}
}
```
