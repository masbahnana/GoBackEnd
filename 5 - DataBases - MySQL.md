# DataBases
## Integração MySQL e Go
Vamos supor que você seja um desenvolvedor e queira salvar os dados do seu aplicativo em um banco de dados MySQL. Como primeiro passo, você tem que estabelecer uma conexão entre o seu aplicativo e o MySQL

- Verifique se o MySQL está instalado e em execução localmente na porta 3306

```$ ps -ef | grep 3306```

- 1.Instale o pacote ```github.com/go-sql-driver/mysql```, usando o comando ```go get```

```$ go get github.com/go-sql-driver/mysql```

- 2.Crie o```connect-mysql.go``` em seguida, nos conectamos ao banco de dados MySQL e realizamos uma consulta SELECT para obter o nome atual do banco de dados

```
package main
import 
(
  "database/sql"
  "fmt"
  "log"
  "net/http"
  "github.com/go-sql-driver/mysql"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  DRIVER_NAME = "mysql"
  DATA_SOURCE_NAME = "root:password@/mydb"
)
var db *sql.DB
var connectionError error
func init() 
{
  db, connectionError = sql.Open(DRIVER_NAME, DATA_SOURCE_NAME)
  if connectionError != nil 
  {
    log.Fatal("error connecting to database :: ", connectionError)
  }
}
func getCurrentDb(w http.ResponseWriter, r *http.Request) 
{
  rows, err := db.Query("SELECT DATABASE() as db")
  if err != nil 
  {
    log.Print("error executing query :: ", err)
    return
  }
  var db string
  for rows.Next() 
  {
    rows.Scan(&db)
  }
  fmt.Fprintf(w, "Current Database is :: %s", db)
}
func main() 
{
  http.HandleFunc("/", getCurrentDb)
  defer db.Close()
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run connect-mysql.go```

### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Navegando para ```http://localhost:8080/``` retornará o nome do banco de dados atual

**Entendendo as partes**

- A importação ```("database/sql" "fmt" "log" "net/http" _ "github.com/go-sql-driver/mysql")```, nós importamos ```github.com/go-sql-driver/mysql``` para o seu efeitos colaterais ou inicialização, usando o sublinhado na frente de uma instrução de importação explicitamente

- Usando ```var db * sql.DB```, declaramos uma instância de banco de dados privada

> Dependendo do tamanho do projeto, você pode declarar uma instância de banco de dados globalmente, injetá-la como uma dependência usando manipuladores ou colocar o ponteiro do conjunto de conexões em ```x/net/context```

- Definimos uma função ```init()``` em que nos conectamos ao banco de dados passando o nome do driver do banco de dados e a fonte de dados para ele

- ```getCurrentDb```, que basicamente executa uma consulta de seleção no banco de dados para obter o nome do banco de dados atual, repetir os registros, copiar seu valor na variável e, eventualmente, gravá-lo em um fluxo de resposta HTTP


##Criando seu primeiro registro no MySQL
Criar ou salvar um registro em um banco de dados nos obriga a escrever consultas SQL e executá-las, **implementar mapeamento relacional de objeto (ORM object-relational mapping)** ou implementar técnicas de mapeamento de dados.
Vamos escrever uma consulta SQL e executá-la usando o pacote ```database / sql``` para criar um registro. Para conseguir isso, você também pode implementar o ORM usando qualquer biblioteca de um número de bibliotecas de terceiros disponíveis no Go, como ```https://github.com/jinzhu/gorm, https://github.com/go-gorp/ gorp``` e ```https://github.com/jirfag/go-queryset```

Como já estabelecemos uma conexão com o banco de dados MySQL em nossa receita anterior, vamos apenas estendê-lo para criar um registro executando uma consulta SQL.

Antes de criar um registro, temos que criar uma tabela no banco de dados MySQL, o que faremos executando os comandos

[print](print.pgn)


- 1.Instale os pacotes ```github.com/go-sql-driver/mysql``` e ```github.com/gorilla/mux```, usando o comando ```go get```

```
$ go get github.com/go-sql-driver/mysql
$ go get github.com/gorilla/mux
```

- 2.Crie create-record-mysql.go. Em seguida, nos conectamos ao banco de dados MySQL e realizamos uma consulta INSERT para criar um registro de funcionário

``` 
package main
import 
(
  "database/sql"
  "fmt"
  "log"
  "net/http"
  "strconv"
  "github.com/go-sql-driver/mysql"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  DRIVER_NAME = "mysql"
  DATA_SOURCE_NAME = "root:password@/mydb"
)
var db *sql.DB
var connectionError error
func init() 
{
  db, connectionError = sql.Open(DRIVER_NAME, DATA_SOURCE_NAME)
  if connectionError != nil 
  {
    log.Fatal("error connecting to database : ", connectionError)
  }
}
func createRecord(w http.ResponseWriter, r *http.Request) 
{
  vals := r.URL.Query()
  name, ok := vals["name"]
  if ok 
  {
    log.Print("going to insert record in database for name : ",
    name[0])
    stmt, err := db.Prepare("INSERT employee SET name=?")
    if err != nil 
    {
      log.Print("error preparing query :: ", err)
      return
    }
    result, err := stmt.Exec(name[0])
    if err != nil 
    {
      log.Print("error executing query :: ", err)
      return
    }
    id, err := result.LastInsertId()
    fmt.Fprintf(w, "Last Inserted Record Id is :: %s",
    strconv.FormatInt(id, 10))
  } 
  else 
  {
    fmt.Fprintf(w, "Error occurred while creating record in 
    database for name :: %s", name[0])
  }
}
func main() 
{
  router := mux.NewRouter()
  router.HandleFunc("/employee/create", createRecord).
  Methods("POST")
  defer db.Close()
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run create-record-mysql.go```