## The application

In the following steps we will create a simple application with [Go](http://golang.org). We start by creating the following `app.go` file:

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

To make sure everything is working as expected we add a few tests by creating a new file `app_test.go` with the following content:

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
