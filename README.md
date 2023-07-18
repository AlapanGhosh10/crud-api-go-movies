# Build A CRUD API With Golang

![10000ft View](https://drive.google.com/uc?export=view&id=1DyQd2qTm_Ejhvn8e3jTHFNHeKr-6CutR)

### Steps

`go mod init project-name`

gorilla mux is an external package

we can install it using → `**go get "github.com/gorilla/mux"**`

- main.go - Basic Setup
    
    ### Movie and Director struct
    
    ```go
    type Movie struct {
    	ID string `json:"id"`
    	Isbn string `json:"isbn"`
    	Title string `json:"title"`
    	Director *Director `json:"director"`
    }
    
    type Director struct {
    	Firstname string `json:"firstname"`
    	Lastname string `json:"lastname"`
    }
    ```
    
    `json:"firstname"` -> these are Field Tags, used for endcoding and decosing json
    
    ### Appending data to Slice of Movie to test the API
    
    ```go
    var movies = []Movie{}
    
    func main() {
    	movies = append(movies, Movie{ID: "1", Isbn:"438227", Title:"Movie One", Director: &Director{Firstname:"John", Lastname: "Doe"}})
    	movies = append(movies, Movie{ID: "2", Isbn: "45455", Title: "Movie Two", Director: &Director{Firstname: "Steve", Lastname: "Harvey"}})
    ```
    
    ### Creating Router
    
    ```go
    r := mux.NewRouter()
    	r.HandleFunc("/movies", getMovies).Methods("GET")
    	r.HandleFunc("/movies/{id}", getMovie).Methods("GET")
    	r.HandleFunc("/movies", createMovie).Methods("POST")
    	r.HandleFunc("/movies/{id}", updateMovie).Methods("PUT")
    	r.HandleFunc("/movies/{id}", deleteMovie).Methods("DELETE")
    ```
    
    ### Starting Server
    
    ```go
    fmt.Printf("Starting server at port 8000\n")
    log.Fatal(http.ListenAndServe(":8000", r))
    ```
    
- getMovies()
    
    ```go
    func getMovies(w http.ResponseWriter, r *http.Request) {
    	w.Header().Set("Content-Type", "application/json")
    	json.NewEncoder(w).Encode(movies)
    }
    ```
    
    w.Header().Set("Content-Type", "application/json") → setting type as json, it is go struct
    
    json.NewEncoder(w).Encode(movies) → encoding movies slice to json and send it in w
    
- deleteMovie()
    
    ```go
    unc deleteMovie(w http.ResponseWriter, r *http.Request) {
    	w.Header().Set("Content-Type", "application/json")
    	params := mux.Vars(r)
    	for index, item := range movies {
    		if item.ID == params["id"] {
    			movies = append(movies[:index], movies[index+1:]...)
    			break
    		}
    	}
    }
    ```
    
    Set encoding type to json
    
    mux.Vars() → returns the route variable for the urrent request → return map[string]string
    
    ### Deleting using append
    
    `**movies = append(movies[:index], movies[index+1:]...)**`
    
    replace the place of the index with the rest of slice from i + 1 to end
    
- getMovie()
    
    ```go
    func getMovie(w http.ResponseWriter, r *http.Request) {
    	w.Header().Set("Content-Type", "application/json")
    	params := mux.Vars(r)
    	for _, item := range movies {
    		if item.ID == params["id"] {
    			json.NewEncoder(w).Encode(item)
    			return
    		}
    	}
    }
    ```
    
- createMovie()
    
    ```go
    func createMovie(w http.ResponseWriter, r *http.Request) {
    	w.Header().Set("Content-Type", "appication/json")
    	var movie Movie
    	_ = json.NewDecoder(r.Body).Decode(&movie)
    	movie.ID = strconv.Itoa(rand.Intn(100000000))
    	movies = append(movies, movie)
    	json.NewEncoder(w).Encode(movies)
    }
    ```
    
    _ = json.NewDecoder(r.Body).Decode(&movie) → decoding json to go struct
    
- updateMovie()
    
    Delete the movie then add it again
    (but this should not be done in the case of a database)
    
    ```go
    func updateMovie(w http.ResponseWriter, r *http.Request) {
    	w.Header().Set("Content-Type", "application/json")
    	params := mux.Vars(r)
    
    	for index, item := range movies {
    		if item.ID == params["id"] {
    			movies = append(movies[:index], movies[index + 1:]...)
    			var movie Movie
    			_ = json.NewDecoder(r.Body).Decode(&movie)
    			movie.ID = params["id"]
    			movies = append(movies, movie)
    			json.NewEncoder(w).Encode(movie)
    		}
    	}
    }
    ```
    

go build

go run main.go

test with Postman
