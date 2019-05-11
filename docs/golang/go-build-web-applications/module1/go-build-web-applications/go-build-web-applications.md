# 构建 web


	package main

	import (
		"database/sql"

		_ "github.com/go-sql-driver/mysql"

		"fmt"
		"log"
		"net/http"
		"os"

		"github.com/gorilla/mux"
	)

	var database *sql.DB

	const (
		PORT    = ":9000"
		DBHost  = "localhost"
		DBPort  = ":3306"
		DBUser  = "wnn"
		DBPass  = "wnnwnn"
		DBDbase = "test"
	)

	type Page struct {
		Title   string
		Content string
		Date    string
	}

	func pageHandler(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r) //解析查询参数为 map
		fmt.Println("%v", vars)
		pageID := vars["id"]
		fileName := "files/" + pageID + ".html"
		_, err := os.Stat(fileName)
		if err != nil {
			fileName = "files/404.html"
		}
		http.ServeFile(w, r, fileName)
	}

	func ServePage(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		pageID := vars["id"]
		thisPage := Page{}
		fmt.Println(pageID)
		err := database.QueryRow("select page_title,page_content,page_date from pages where id=?", pageID).Scan(&thisPage.Title, &thisPage.Content, &thisPage.Date)
		if err != nil {
			log.Println("Couldn't get page:" + pageID)
			log.Println(err.Error)
		}
		html := `<html><head><title>` + thisPage.Title +
			`</title></head><body><h1>` + thisPage.Title + `</h1><div>` +
			thisPage.Content + `</div></body></html>`
		fmt.Fprintln(w, html)
	}

	func ServePageByGUID(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		pageGUID := vars["guid"]
		thisPage := Page{}
		fmt.Println(pageGUID)
		err := database.QueryRow("select page_title,page_content,page_date from pages where page_guid=?", pageGUID).Scan(&thisPage.Title, &thisPage.Content, &thisPage.Date)
		if err != nil {
			log.Println("Couldn't get page:" + pageGUID)
			http.Error(w, http.StatusText(404), http.StatusNotFound)
			log.Println(err.Error)
			return // 这里应该需要 return，书里没有
		}
		html := `<html><head><title>` + thisPage.Title +
			`</title></head><body><h1>` + thisPage.Title + `</h1><div>` +
			thisPage.Content + `</div></body></html>`
		fmt.Fprintln(w, html)
	}

	func main() {
		dbConn := fmt.Sprintf("%s:%s@/%s", DBUser, DBPass, DBDbase)
		fmt.Println(dbConn)
		db, err := sql.Open("mysql", dbConn)
		if err != nil {
			log.Println("Couldn't connect to: " + DBDbase)
			log.Println(err.Error)
		}
		database = db

		rtr := mux.NewRouter()
		// rtr.HandleFunc("/pages/{id:[0-9]+}", ServePage)
		rtr.HandleFunc("/pages/{guid:[a-z0-9A\\-]+}", ServePageByGUID)
		http.Handle("/", rtr)
		http.ListenAndServe(PORT, nil)
	}


# 1 An Introduction to Concurrency in Go

注意引用传递在 defer 中的坑。下边的例子输出0而不是100

```go
package main

import "fmt"

func main() {
	a := new(int)

	defer fmt.Println(*a)

	for i := 0; i < 100; i++ {
		*a++
	}
}
```

### channel 的使用。 CSP(Communicating Sequential Processes)

```go
package main

import (
	"fmt"
	"strings"
	"sync"
)

var finalString string
var initString string
var stringLen int

func addToFinalString(letterChan chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	letter := <-letterChan
	finalString += letter
}

func upper(letterChan chan string, letter string, wg *sync.WaitGroup) {
	defer wg.Done()
	upperLetter := strings.ToUpper(letter)
	letterChan <- upperLetter
}

func main() {
	var wg sync.WaitGroup
	initString = "gebilaowang"
	initBytes := []byte(initString)
	stringLen = len(initBytes)
	var letterChan chan string = make(chan string)
	for i := 0; i < stringLen; i++ {
		wg.Add(2)

		go upper(letterChan, string(initBytes[i]), &wg)
		go addToFinalString(letterChan, &wg)

		wg.Wait()
	}
	fmt.Println(finalString)
}
```

### select 的使用

语法类似 switch 。
select reacts to actions and communication across a channel.

```go
switch {
	case 'x':
	case 'y':
}
select {
	case <- channelA:
	case <- channelB:
}
```

select 会 block直到有数据发送给channel。否则会 deadlocks
如果同时 receive，go 会无法预期地执行其中一个

```go
package main

import (
	"fmt"
	"strings"
	"sync"
)

var initialString string
var initialBytes []byte
var stringLength int
var finalString string
var lettersProcessed int
var wg sync.WaitGroup
var applicationStatus bool

func getLetters(gQ chan string) {

	for i := range initialBytes {
		gQ <- string(initialBytes[i])
	}

}

func capitalizeLetters(gQ chan string, sQ chan string) {

	for {
		if lettersProcessed >= stringLength {
			applicationStatus = false
			break
		}
		select {
		case letter := <-gQ:
			capitalLetter := strings.ToUpper(letter)
			finalString += capitalLetter
			lettersProcessed++
		}
	}
}

func main() {

	applicationStatus = true

	getQueue := make(chan string)
	stackQueue := make(chan string)

	initialString = "Four score and seven years ago our fathers brought forth on this continent, a new nation, conceived in Liberty, and dedicated to the proposition that all men are created equal."
	initialBytes = []byte(initialString)
	stringLength = len(initialString)
	lettersProcessed = 0

	fmt.Println("Let's start capitalizing")

	go getLetters(getQueue)
	capitalizeLetters(getQueue, stackQueue)

	close(getQueue)
	close(stackQueue)

	for {

		if applicationStatus == false {
			fmt.Println("Done")
			fmt.Println(finalString)
			break
		}

	}
}
```