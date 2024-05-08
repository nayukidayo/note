```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/api", cors(api))
	log.Fatalln(http.ListenAndServe(":3000", nil))
}

func cors(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		if origin := r.Header.Get("Origin"); origin != "" {
			w.Header().Set("Access-Control-Allow-Credentials", "true")
			w.Header().Set("Access-Control-Allow-Origin", r.Header.Get("Origin"))
			w.Header().Set("Vary", "Origin")
		}
		if r.Method == http.MethodOptions {
			w.Header().Set("Access-Control-Allow-Methods", r.Header.Get("Access-Control-Request-Method"))
			w.Header().Set("Access-Control-Allow-Headers", r.Header.Get("Access-Control-Request-Headers"))
			w.Header().Set("Access-Control-Max-Age", "86400")
			w.WriteHeader(http.StatusNoContent)
			return
		}
		next(w, r)
	}
}

func api(w http.ResponseWriter, r *http.Request) {
	fmt.Println("[cookies]", r.Cookies())

	w.Header().Set("foo", "123")
	w.Header().Set("Access-Control-Expose-Headers", "foo")

	data, _ := json.Marshal(map[string]string{"bar": "123"})
	w.Header().Set("Content-Type", "application/json")
	w.Write(data)
}
```
