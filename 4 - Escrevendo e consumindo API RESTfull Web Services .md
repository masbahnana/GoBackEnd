# Escrevendo e consumindo API RESTfull Web Services 
Sempre que construímos um aplicativo da Web que encapsula a lógica que pode ser útil para outros aplicativos relacionados, muitas vezes também escrevemos e consumimos serviços da web. Isso ocorre porque eles expõem a funcionalidade em uma rede, que é acessível através do protocolo HTTP, tornando um aplicativo uma única fonte de verdade

## GET
Ao escrever aplicativos da web, geralmente temos que expor nossos serviços ao cliente ou à interface do usuário para que eles consumam uma parte do código em execução em um sistema diferente. Expor o serviço pode ser feito com métodos de protocolo HTTP. Dos muitos métodos HTTP, estaremos aprendendo a implementar o método HTTP GET nesta receita

- 1.Instale o pacote ```github.com/gorilla/mux``` com o ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-rest-get.go``` onde definiremos duas rotas - ```/employees``` e ```/employee/{id}``` junto com seus manipuladores. O primeiro grava o array estático de funcionários e o segundo grava detalhes do funcionário para o ID fornecido para um fluxo de resposta HTTP

```
package main
import 
(
  "encoding/json"
  "log"
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Route struct 
{
  Name string
  Method string
  Pattern string
  HandlerFunc http.HandlerFunc
}
type Routes []Route
var routes = Routes
{
  Route
  {
    "getEmployees",
    "GET",
    "/employees",
    getEmployees,
  },
  Route
  {
    "getEmployee",
    "GET",
    "/employee/{id}",
    getEmployee,
  },
}
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
type Employees []Employee
var employees []Employee
func init() 
{
  employees = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  json.NewEncoder(w).Encode(employees)
}
func getEmployee(w http.ResponseWriter, r *http.Request) 
{
  vars := mux.Vars(r)
  id := vars["id"]
  for _, employee := range employees 
  {
    if employee.Id == id 
    {
      if err := json.NewEncoder(w).Encode(employee); err != nil 
      {
        log.Print("error getting requested employee :: ", err)
      }
    }
  }
}
func AddRoutes(router *mux.Router) *mux.Router 
{
  for _, route := range routes 
  {
    router.
    Methods(route.Method).
    Path(route.Pattern).
    Name(route.Name).
    Handler(route.HandlerFunc)
  }
  return router
}
func main() 
{
  muxRouter := mux.NewRouter().StrictSlash(true)
  router := AddRoutes(muxRouter)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
``` 

- 3.Rode no terminal

```$ go run http-rest-get.go``` 


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Em seguida, executar uma solicitação **GET** a partir da linha de comando da seguinte maneira fornecerá uma lista de todos os funcionários

```$ curl -X GET http://localhost:8080/employees[{"id":"1","firstName":"Foo","lastName":"Bar"},{"id":"2","firstName":"Baz","lastName":"Qux"}]```


- Executar uma solicitação GET para um ID de funcionário específico da linha de comando da seguinte maneira, fornecerá os detalhes do funcionário para o ID correspondente

```$ curl -X GET http://localhost:8080/employee/1 {"id":"1","firstName":"Foo","lastName":"Bar"}```

**Entendo as partes**
- Usamos ```import ("enconding/json" "log" "net/http" "strconv" "github.com/gorilla/mux")```. Aqui, nós importamos o ```github.com/gorilla/mux``` para criar um Gorilla Mux Router

- Em seguida, declaramos o tipo de ```struct Route``` com quatro campos - ```Name, Method, Pattern e HandlerFunc```, onde Name representa o nome de um método HTTP, Method representa o tipo de método HTTP que pode ser ```GET, POST, PUT, DELETE``` e assim on, Pattern representa o caminho da URL e HandlerFunc representa o manipulador HTTP

- Em seguida, definimos duas rotas para a solicitação GET

```
var routes = Routes
{
  Route
  {
    "getEmployees",
    "GET",
    "/employees",
    getEmployees,
  },
  Route
  {
    "getEmployee",
    "GET",
    "/employee/{id}",
    getEmployee,
  },
}
```
- Definimos um array de empregados estáticos

```
func init() 
{
  employees = Employees 
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
}
```

- Em seguida, definimos dois manipuladores - ```getEmployees``` e ```getEmployee```, em que o primeiro apenas organiza uma matriz estática de funcionários e os grava em um fluxo de resposta HTTP, e o último obtém o ID de funcionário de uma variável de solicitação HTTP, busca o funcionário para o ID correspondente o array, empacota o objeto e o grava em um fluxo de resposta HTTP

- Seguindo os manipuladores, definimos uma função ```AddRoutes```, que itera sobre a matriz de rotas que definimos, a adiciona ao roteador ```gorilla / mux``` e retorna o objeto Router

- Finalmente, definimos ```main ()``` onde criamos uma instância do roteador ```gorilla / mux``` usando o manipulador ```NewRouter ()``` com o comportamento de barra para novas rotas como true, o que significa que o aplicativo sempre verá o caminho conforme especificado na rota. Por exemplo, se o caminho da rota for ```/ path /```, o acesso ```/path``` será redirecionado para o primeiro e vice-versa


## POST
Sempre que temos que enviar dados para o servidor através de uma chamada assíncrona ou através de um formulário HTML, então vamos com a implementação do método HTTP POST

- 1.Instale o pacote ```github.com/gorilla/mux``` com o ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-rest-post.go``` onde definiremos uma rota adicional que suporte o método HTTP ```POST``` e um manipulador que adiciona um funcionário à matriz estática inicial de funcionários e grava a lista atualizada em um fluxo de resposta HTTP

```
package main
import 
(
  "encoding/json"
  "log"
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Route struct 
{
  Name string
  Method string
  Pattern string
  HandlerFunc http.HandlerFunc
}
type Routes []Route
var routes = Routes
{
  Route
  {
    "getEmployees",
    "GET",
    "/employees",
    getEmployees,
  },
  Route
  {
    "addEmployee",
    "POST",
    "/employee/add",
    addEmployee,
  },
}
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
type Employees []Employee
var employees []Employee
func init() 
{
  employees = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  json.NewEncoder(w).Encode(employees)
}
func addEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  err := json.NewDecoder(r.Body).Decode(&employee)
  if err != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", err)
    return
  }
  log.Printf("adding employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  employees = append(employees, Employee{Id: employee.Id, 
  FirstName: employee.FirstName, LastName: employee.LastName})
  json.NewEncoder(w).Encode(employees)
}
func AddRoutes(router *mux.Router) *mux.Router 
{
  for _, route := range routes 
  {
    router.
    Methods(route.Method).
    Path(route.Pattern).
    Name(route.Name).
    Handler(route.HandlerFunc)
  }
  return router
}
func main() 
{
  muxRouter := mux.NewRouter().StrictSlash(true)
  router := AddRoutes(muxRouter)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-rest-post.go```

### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ``` 8080``` 

- A execução de uma solicitação ``` POST```  a partir da linha de comando da seguinte maneira adicionará um funcionário à lista com ``` ID como 3```  e retornará a lista de funcionários como uma resposta

```$ curl -H "Content-Type: application/json" -X POST -d '{"Id":"3", "firstName":"Quux", "lastName":"Corge"}' http://localhost:8080/employee/add``` 

**Entendendo as partes**
- Adicionamos outra rota com o nome addEmployee que executa o manipulador addEmployee para cada solicitação POST para o padrão de URL ```/employee/add``` 

- Definimos um manipulador ``` addEmployee```, que basicamente decodifica os dados do funcionário que fazem parte de uma solicitação POST usando o manipulador ```NewDecoder``` do pacote ```encoding/ son``` integrado do Go, anexa-o à matriz estática inicial de um funcionário e grava-o em um fluxo de resposta HTTP


## PUT
Sempre que quisermos atualizar um registro que tenhamos criado anteriormente ou quisermos criar um novo registro se ele não existir, muitas vezes denominado Upsert, então vamos com a implementação do método HTTP PUT

- 1.Instale o pacote ```github.com/gorilla/mux``` com o ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-rest-put.go``` onde definiremos uma rota adicional que suporte o método HTTP ```PUT``` e um manipulador que atualize os detalhes do funcionário para a ID fornecida ou adicione um funcionário à matriz estática inicial de funcionários; se o ID não existir, empacote-o no JSON e grave-o em um fluxo de resposta HTTP

```
package main
import 
(
  "encoding/json"
  "log"
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Route struct 
{
  Name string
  Method string
  Pattern string
  HandlerFunc http.HandlerFunc
}
type Routes []Route
var routes = Routes
{
  Route
  {
    "getEmployees",
    "GET",
    "/employees",
    getEmployees,
  },
  Route
  {
    "addEmployee",
    "POST",
    "/employee/add",
    addEmployee,
  },
  Route
  {
    "updateEmployee",
    "PUT",
    "/employee/update",
    updateEmployee,
  },
}
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
type Employees []Employee
var employees []Employee
func init() 
{
  employees = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  json.NewEncoder(w).Encode(employees)
}
func updateEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  err := json.NewDecoder(r.Body).Decode(&employee)
  if err != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", err)
    return
  }
  var isUpsert = true
  for idx, emp := range employees 
  {
    if emp.Id == employee.Id 
    {
      isUpsert = false
      log.Printf("updating employee id :: %s with 
      firstName as :: %s and lastName as:: %s ", 
      employee.Id, employee.FirstName, employee.LastName)
      employees[idx].FirstName = employee.FirstName
      employees[idx].LastName = employee.LastName
      break
    }
  }
  if isUpsert 
  {
    log.Printf("upserting employee id :: %s with 
    firstName as :: %s and lastName as:: %s ", 
    employee.Id, employee.FirstName, employee.LastName)
    employees = append(employees, Employee{Id: employee.Id,
    FirstName: employee.FirstName, LastName: employee.LastName})
  }
  json.NewEncoder(w).Encode(employees)
}
func addEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  err := json.NewDecoder(r.Body).Decode(&employee)
  if err != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", err)
    return
  }
  log.Printf("adding employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  employees = append(employees, Employee{Id: employee.Id, 
  FirstName: employee.FirstName, LastName: employee.LastName})
  json.NewEncoder(w).Encode(employees)
}
func AddRoutes(router *mux.Router) *mux.Router 
{
  for _, route := range routes 
  {
    router.
    Methods(route.Method).
    Path(route.Pattern).
    Name(route.Name).
    Handler(route.HandlerFunc)
  }
  return router
}
func main() 
{
  muxRouter := mux.NewRouter().StrictSlash(true)
  router := AddRoutes(muxRouter)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-rest-put.go``` 


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```

- Executar uma solicitação ```PUT``` a partir da linha de comando da seguinte maneira, atualizará o ```firstName``` e o ```lastName``` para um funcionário com ID 1

```$ curl -H "Content-Type: application/json" -X PUT -d '{"Id":"1", "firstName":"Grault", "lastName":"Garply"}' http://localhost:8080/employee/update```

- Execute uma solicitação ```PUT``` para um funcionário com ID 3 na linha de comando da seguinte maneira, ele adicionará outro funcionário à matriz, pois não há nenhum funcionário com ID 3, demonstrando o cenário upsert

```$ curl -H "Content-Type: application/json" -X PUT -d '{"Id":"3", "firstName":"Quux", "lastName":"Corge"}' http://localhost:8080/employee/update```

**Entendendo as partes**
- Primeiro, adicionamos outra rota com o nome ```updateEmployee```, que executa o manipulador ```updateEmployee``` para cada solicitação PUT para o padrão de URL ```/employee/update```

- ```updateEmployee```, que basicamente decodifica os dados do funcionário que fazem parte de uma solicitação ```PUT``` usando o manipulador ```NewDecoder``` do pacote ```encoding / json``` interno de Go, repete a matriz employees para saber se o ID do funcionário solicitado existe na matriz estática inicial de funcionários, que também podemos chamar de cenário UPDATE ou UPSERT, executa a ação necessária e grava a resposta em um fluxo de resposta HTTP


## DELETE
Sempre que queremos remover um registro que não é mais necessário, então vamos com a implementação do método HTTP DELETE

- 1.Instale o pacote ```github.com/gorilla/mux``` com o ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-rest-delete.go``` onde definiremos uma rota que suporte o método HTTP ```DELETE``` e um manipulador que exclua os detalhes do funcionário para a ID fornecida da matriz estática de funcionários, empacote a matriz para JSON e grave-a em um Fluxo de resposta HTTP

```
package main
import 
(
  "encoding/json"
  "log"
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Route struct 
{
  Name string
  Method string
  Pattern string
  HandlerFunc http.HandlerFunc
}
type Routes []Route
var routes = Routes
{
  Route
  {
    "getEmployees",
    "GET",
    "/employees",
    getEmployees,
  },
  Route
  {
    "addEmployee",
    "POST",
    "/employee/add/",
    addEmployee,
  },
  Route
  {
    "deleteEmployee",
    "DELETE",
    "/employee/delete",
    deleteEmployee,
  },
}
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
type Employees []Employee
var employees []Employee
func init() 
{
  employees = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  json.NewEncoder(w).Encode(employees)
}
func deleteEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  err := json.NewDecoder(r.Body).Decode(&employee)
  if err != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", err)
    return
  }
  log.Printf("deleting employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  index := GetIndex(employee.Id)
  employees = append(employees[:index], employees[index+1:]...)
  json.NewEncoder(w).Encode(employees)
}
func GetIndex(id string) int 
{
  for i := 0; i < len(employees); i++ 
  {
    if employees[i].Id == id 
    {
      return i
    }
  }
  return -1
}
func addEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  err := json.NewDecoder(r.Body).Decode(&employee)
  if err != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", err)
    return
  }
  log.Printf("adding employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  employees = append(employees, Employee{Id: employee.Id, 
  FirstName: employee.FirstName, LastName: employee.LastName})
  json.NewEncoder(w).Encode(employees)
}
func AddRoutes(router *mux.Router) *mux.Router 
{
  for _, route := range routes 
  {
    router.
    Methods(route.Method).
    Path(route.Pattern).
    Name(route.Name).
    Handler(route.HandlerFunc)
  }
  return router
}
func main() 
{
  muxRouter := mux.NewRouter().StrictSlash(true)
  router := AddRoutes(muxRouter)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-rest-delete.go``` 

