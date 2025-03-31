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
	http.HandleFunc("GET /live", handleLive())
	http.HandleFunc("GET /{file...}", handleFS())
	log.Fatalln(http.ListenAndServe(":3000", nil))
}

func handleLive() http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "text/event-stream")
		w.Header().Set("Cache-Control", "no-cache")
		w.Header().Set("X-Accel-Buffering", "no")

		rc := http.NewResponseController(w)

		for range 10 {
			w.Write([]byte("event: "))
			w.Write([]byte("hi"))
			w.Write([]byte("\n"))
			
			w.Write([]byte("data: "))
			w.Write([]byte("hello"))
			w.Write([]byte("\n\n"))

			if err := rc.Flush(); err != nil {
				break
			}
		}
	}
}

func handleFS() http.HandlerFunc {
	dist, _ := ui.DistFS()
	return func(w http.ResponseWriter, r *http.Request) {
		var cc string
		f, err := dist.Open(r.PathValue("file"))
		if err == nil {
			f.Close()
			cc = "max-age=1209600, stale-while-revalidate=86400"
		} else {
			r.URL.Path = "/"
			cc = "no-cache"
		}
		w.Header().Set("Cache-Control", cc)
		http.FileServerFS(dist).ServeHTTP(w, r)
	}
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
