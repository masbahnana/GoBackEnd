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



## Criando seu primeiro registro no MySQL
Criar ou salvar um registro em um banco de dados nos obriga a escrever consultas SQL e executá-las, **implementar mapeamento relacional de objeto (ORM object-relational mapping)** ou implementar técnicas de mapeamento de dados.
Vamos escrever uma consulta SQL e executá-la usando o pacote ```database / sql``` para criar um registro. Para conseguir isso, você também pode implementar o ORM usando qualquer biblioteca de um número de bibliotecas de terceiros disponíveis no Go, como ```https://github.com/jinzhu/gorm, https://github.com/go-gorp/ gorp``` e ```https://github.com/jirfag/go-queryset```

Como já estabelecemos uma conexão com o banco de dados MySQL em nossa receita anterior, vamos apenas estendê-lo para criar um registro executando uma consulta SQL.

Antes de criar um registro, temos que criar uma tabela no banco de dados MySQL, o que faremos executando os comandos

![imagem](/print.png)


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

### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Executar uma solicitação ```POST``` para criar um registro de funcionário na linha de comando da seguinte maneira fornecerá o ID do último registro criado

```
$ curl -X POST http://localhost:8080/employee/create?name=foo
Last created record id is :: 1
```

**Entendendo as partes**
- ```import ("banco de dados / sql" "fmt" "log" "net / http" "strconv" _ "github.com/go-sql-driver/mysql" "github.com/gorilla/mux")```, nós importamos o``` github.com/gorilla/mux```para criar um Gorilla Mux Router e inicializar o driver Go MySQL, importando o pacote ```github.com/go-sql-driver/mysql```

- Em seguida, definimos um manipulador ```createRecord```, que busca o nome da solicitação, atribui-o ao nome da variável local, prepara uma instrução INSERT com um espaço reservado para nome que será substituído dinamicamente pelo nome, executa a instrução e, por fim, grava a última ID criado para um fluxo de resposta HTTP


## Lendo registros do MySQL
Antes nós criamos um registro de funcionário no banco de dados MySQL. Agora aprenderemos como podemos lê-lo executando uma consulta SQL

- 1.Instale os pacotes ```github.com/go-sql-driver/mysql``` e ```github.com/gorilla/mux``` usando o comando ```go get```

```
$ go get github.com/go-sql-driver/mysql
$ go get github.com/gorilla/mux
```

- 2.Crie ```read-record-mysql.go``` onde nos conectamos ao banco de dados MySQL, realizamos uma consulta ```SELECT``` para obter todos os funcionários do banco de dados, iterar os registros, copiar seu valor para a estrutura, adicionar todos eles a uma lista e gravá-lo em um fluxo de resposta HTTP

```
package main
import 
(
  "database/sql" "encoding/json"
  "log"
  "net/http"
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
    log.Fatal("error connecting to database :: ", connectionError)
  }
}
type Employee struct 
{
  Id int `json:"uid"`
  Name string `json:"name"`
}
func readRecords(w http.ResponseWriter, r *http.Request) 
{
  log.Print("reading records from database")
  rows, err := db.Query("SELECT * FROM employee")
  if err != nil 
  {
    log.Print("error occurred while executing select 
    query :: ",err)
    return
  }
  employees := []Employee{}
  for rows.Next() 
  {
    var uid int
    var name string
    err = rows.Scan(&uid, &name)
    employee := Employee{Id: uid, Name: name}
    employees = append(employees, employee)
  }
  json.NewEncoder(w).Encode(employees)
}
func main() 
{
  router := mux.NewRouter()
  router.HandleFunc("/employees", readRecords).Methods("GET")
  defer db.Close()
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run read-record-mysql.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```

- Navegando para ```http://localhost:8080/employees``` listará todos os registros da tabela de funcionários

**Entendendo as partes**
- Usando a importação ```("database/sql" "codificação/json" "log" "net/http" _ "github.com/go-sql-driver/mysql" "github.com/gorilla/mux")```, nós importamos um adicional package, ```encoding / json```, que ajuda a empacotar a estrutura de dados Go para JSON

- Em seguida, declaramos a estrutura de dados Go Person, que possui os campos ```Id``` e ```Name```

> Lembre-se de que o nome do campo deve começar com uma letra maiúscula na definição de tipo ou pode haver erros

- ```readRecords```, que consulta o banco de dados para obter todos os registros da tabela de funcionários, itera os registros, copia seu valor para a estrutura, adiciona todos os registros a uma lista, empacota a lista de objetos em JSON e grava-o em um fluxo de resposta HTTP

## Atualizando seu primeiro registro no MySQL
Você criou um registro para um funcionário em um banco de dados com todos os detalhes, como nome, departamento, endereço e assim por diante, e depois de algum tempo, o funcionário muda de departamento. Nesse caso, temos que atualizar seu departamento em um banco de dados para que seus detalhes fiquem sincronizados por toda a organização, o que pode ser obtido usando uma instrução SQL UPDATE e, nesta receita, aprenderemos como podemos implementá-lo no Go

- 1.Instale os pacotes ```github.com/go-sql-driver/mysql``` e ```github.com/gorilla/mux``` usando o comando ```go get```

```
$ go get github.com/go-sql-driver/mysql
$ go get github.com/gorilla/mux
```

- 2.Crie ```update-record-mysql.go```. Em seguida, nos conectamos ao banco de dados MySQL, atualizamos o nome de um funcionário para um ID e gravamos o número de registros atualizados em um banco de dados em um fluxo de resposta HTTP

```
package main
import 
(
  "database/sql"
  "fmt"
  "log"
  "net/http" 
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
    log.Fatal("error connecting to database :: ", connectionError)
  }
}
type Employee struct 
{
  Id   int    `json:"uid"`
  Name string `json:"name"`
}
func updateRecord(w http.ResponseWriter, r *http.Request) 
{
  vars := mux.Vars(r)
  id := vars["id"]
  vals := r.URL.Query()
  name, ok := vals["name"]
  if ok 
  {
    log.Print("going to update record in database 
    for id :: ", id)
    stmt, err := db.Prepare("UPDATE employee SET name=? 
    where uid=?")
    if err != nil 
    {
      log.Print("error occurred while preparing query :: ", err)
      return
    }
    result, err := stmt.Exec(name[0], id)
    if err != nil 
    {
      log.Print("error occurred while executing query :: ", err)
      return
    }
    rowsAffected, err := result.RowsAffected()
    fmt.Fprintf(w, "Number of rows updated in database 
    are :: %d",rowsAffected)
  } 
  else 
  {
    fmt.Fprintf(w, "Error occurred while updating record in 
    database for id :: %s", id)
  }
}
func main() 
{
  router := mux.NewRouter()
  router.HandleFunc("/employee/update/{id}",
  updateRecord).Methods("PUT")
  defer db.Close()
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run update-record-mysql.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```

- Executar uma solicitação ```PUT``` a partir da linha de comando para atualizar um registro de funcionário com o ID como 1 fornecerá o número de registros atualizados no banco de dados como uma resposta

```
$ curl -X PUT http://localhost:8080/employee/update/1?name\=bar
Number of rows updated in database are :: 1
```

**Entendendo as partes**
- Definimos um manipulador ```updateRecord```, que obtém o ID a ser atualizado no banco de dados como um caminho de variável de caminho de URL e o novo nome como a variável de solicitação, prepara uma instrução de atualização com um nome e UID como um espaço reservado, que será substituído dinamicamente , executa a instrução, obtém o número de linhas atualizadas como resultado de sua execução e as grava em um fluxo de resposta HTTP

- Em seguida, registramos um manipulador ```updateRecord``` a ser chamado para o padrão de ```URL/employee/update/{id}``` para cada solicitação ```PUT``` com o roteador gorilla mux e fechamos o banco de dados usando a instrução ```defer db.Close()``` assim que retornamos do função principal


## Excluindo seu primeiro registro do MySQL
Vamos imaginar que um funcionário deixou a organização e você deseja revogar seus detalhes do banco de dados. Nesse caso, podemos usar a instrução ```SQL DELETE```

- 1.Instale os pacotes ```github.com/go-sql-driver/mysql``` e ```github.com/gorilla/mux```, usando o comando ```go get```

```
$ go get github.com/go-sql-driver/mysql
$ go get github.com/gorilla/mux
```

- 2.Excluindo seu primeiro registro do MySQL. Se você tem um cenário em que um funcionário deixou uma organização e você deseja que seus detalhes tenham um banco de dados. Nesse caso, podemos usar uma instrução SQL DELETE

```
package main
import 
(
  "database/sql"
  "fmt"
  "log"
  "net/http"
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
    log.Fatal("error connecting to database :: ", connectionError)
  }
}
func deleteRecord(w http.ResponseWriter, r *http.Request) 
{
  vals := r.URL.Query()
  name, ok := vals["name"]
  if ok 
  {
    log.Print("going to delete record in database for 
    name :: ", name[0])
    stmt, err := db.Prepare("DELETE from employee where name=?")
    if err != nil 
    {
      log.Print("error occurred while preparing query :: ", err)
      return
    }
    result, err := stmt.Exec(name[0])
    if err != nil 
    {
      log.Print("error occurred while executing query :: ", err)
      return
    }
    rowsAffected, err := result.RowsAffected()
    fmt.Fprintf(w, "Number of rows deleted in database are :: %d",
    rowsAffected)
  } 
  else 
  {
    fmt.Fprintf(w, "Error occurred while deleting record in 
    database for name %s", name[0])
  }
}
func main() 
{
  router := mux.NewRouter()
  router.HandleFunc("/employee/delete",
  deleteRecord).Methods("DELETE")
  defer db.Close()
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run delete-record-mysql.go```


### Como isso deve funcionar
- O servidor HTTP começará a escutar localmente na porta 8080

- Executar uma solicitação DELETE da linha de comando para excluir um funcionário com o nome como barra fornecerá o número de registros excluídos do banco de dados

```
$ curl -X DELETE http://localhost:8080/employee/delete?name\=bar
Number of rows deleted in database are :: 1 
```

**Entendendo as partes**
- ```deleteRecord```, que obtém o nome a ser excluído do banco de dados como a variável request, prepara uma instrução ```DELETE``` com um nome como um espaço reservado, que será substituído dinamicamente, executa a instrução, obtém a contagem de linhas excluídas resultado de sua execução e grava-o em um fluxo de resposta HTTP

- registramos um manipulador deleteRecord a ser chamado para o padrão de URL ```/employee/delete``` para cada solicitação ```DELETE``` com o roteador ```gorilla/mux``` e fechamos o banco de dados usando a instrução ```defer db.Close()``` assim que retornamos da função ```main()```