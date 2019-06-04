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

### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080

- A execução de uma solicitação DELETE da linha de comando da seguinte maneira excluirá um funcionário com ID 1 e nos fornecerá a lista atualizada de funcionários

```$ curl -H "Content-Type: application/json" -X DELETE -d '{"Id":"1", "firstName": "Foo", "lastName": "Bar"}' http://localhost:8080/employee/delete```

**Ententendo as partes**
- Adicionamos outra rota com o nome ```deleteEmployee```, que executa o manipulador ```deleteEmployee``` para cada solicitação ```DELETE``` para o padrão de URL ```/employee/delete```

- ```deleteEmployee```, que basicamente decodifica os dados do funcionário que fazem parte de uma solicitação ```DELETE``` usando o manipulador ```NewDecoder``` do pacote ```encoding / json``` interno de Go, obtém o índice do empregado solicitado usando a função auxiliar ```GetIndex``` , exclui o funcionário e grava o array atualizado como JSON em um fluxo de resposta HTTP


## Versionando a API REST
Quando você cria uma API RESTful para veicular um cliente interno, provavelmente não precisa se preocupar com a versão da sua API. Levando as coisas um passo adiante, se você tiver controle sobre todos os clientes que acessam sua API, o mesmo pode ser verdade.

No entanto, em um caso em que você tem uma API pública ou uma API em que você não tem controle sobre todos os clientes que a usam, a versão da sua API pode ser necessária

- 1.Instale o pacote ```github.com/gorilla/mux``` com o ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-rest-versioning.go``` onde definiremos duas versões do mesmo caminho de URL que suportam o método HTTP GET, com um tendo v1 como prefixo e o outro com v2 como prefixo na rota

```
package main
import 
(
  "encoding/json"
  "log"
  "net/http"
  "strings"
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
}
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
type Employees []Employee
var employees []Employee
var employeesV1 []Employee
var employeesV2 []Employee
func init() 
{
  employees = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
  }
  employeesV1 = Employees
  {
    Employee{Id: "1", FirstName: "Foo", LastName: "Bar"},
    Employee{Id: "2", FirstName: "Baz", LastName: "Qux"},
  }
  employeesV2 = Employees
  {
    Employee{Id: "1", FirstName: "Baz", LastName: "Qux"},
    Employee{Id: "2", FirstName: "Quux", LastName: "Quuz"},
  }
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  if strings.HasPrefix(r.URL.Path, "/v1") 
  {
    json.NewEncoder(w).Encode(employeesV1)
  } 
  else if strings.HasPrefix(r.URL.Path, "/v2") 
  {
    json.NewEncoder(w).Encode(employeesV2)
  } 
  else 
  {
    json.NewEncoder(w).Encode(employees)
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
  // v1
  AddRoutes(muxRouter.PathPrefix("/v1").Subrouter())
  // v2
  AddRoutes(muxRouter.PathPrefix("/v2").Subrouter())
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-rest-versioning.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```

- Executar uma solicitação ```GET``` com o prefixo do caminho como / v1 a partir da linha de comando da seguinte maneira, fornecerá uma lista de um conjunto de funcionários

```$ curl -X GET http://localhost:8080/v1/employees [{"id":"1","firstName":"Foo","lastName":"Bar"},{"id":"2","firstName":"Baz","lastName":"Qux"}] ```

- Executar uma solicitação GET com prefixo de caminho como / v2 fornecerá uma lista de outro conjunto de funcionários

```$ curl -X GET http://localhost:8080/v2/employees [{"id":"1","firstName":"Baz","lastName":"Qux"},{"id":"2","firstName":"Quux","lastName":"Quuz"}]```

- Ao projetar a URL REST, preferimos retornar os dados padrão se o cliente consultar o terminal sem especificar a versão no caminho da URL. Para incorporá-lo, modificamos o manipulador getEmployees para verificar o prefixo na URL e agir de acordo. Assim, executar uma solicitação GET sem o prefixo de caminho da linha de comando, como a seguir, fornecerá uma lista com um único registro, que podemos chamar de resposta padrão ou inicial do ponto de extremidade REST chamado

```$ curl -X GET http://localhost:8080/employees  [{"id":"1","firstName":"Foo","lastName":"Bar"}]```

**Entendendo as partes**
- Primeiro, definimos uma única rota com o nome ```getEmployees```, que executa um manipulador ```getEmployees``` para cada solicitação ```GET``` para o padrão de URL ```/employees```

- Em seguida, criamos três matrizes, ou seja, ```employees, employeesV1 e employeesV2```, que são retornados como uma resposta a uma chamada HTTP ```GET``` para os padrões de ```URL / employees,/v1/employeese/v2/employees``` respectivamente

- Em seguida, definimos um manipulador de ```getEmployees``` onde verificamos o prefixo no caminho da URL e executamos uma ação com base nele

- Em seguida, definimos uma função auxiliar ```AddRoutes```, que repete a matriz de rotas que definimos, a adiciona ao roteador ```gorilla / mux``` e retorna o objeto Router

- Finalmente, definimos ```main ()``` onde criamos uma instância de roteador ```gorilla / mux``` usando o manipulador ```NewRouter ()``` com o comportamento de barra para novas rotas como true e adicionamos rotas a ele chamando a função auxiliar AddRoutes passando o roteador padrão e dois sub-roteadores , um com o prefixo como v1 e o outro com o prefixo como v2


## Criando seu primeiro cliente REST
Hoje, a maioria dos aplicativos que se comunicam com servidores usam serviços RESTful. escreveremos um cliente REST usando o pacote https://gopkg.in/resty.v1, que é inspirado pelo cliente de REST Ruby para consumir os serviços RESTful

- 1.Execute ```http-rest-get.go```, que criamos em uma das nossas receitas anteriores, em um terminal separado, executando o seguinte comando

```$ go run http-rest-get.go```

- 2.Execute http-rest-get.go, que criamos anteriormente

- 3.Em um terminal separado, executando o seguinte comando

```$ curl -X GET http://localhost:8080/employees```

- 4.Isso deve retornar a seguinte resposta

```[{"id":"1","firstName":"Foo","lastName":"Bar"},{"id":"2","firstName":"Baz","lastName":"Qux"}]```
- 5.Instale os pacotes ```github.com/gorilla/mux``` e ```gopkg.in/resty.v1``` usando o comando ```go get```

```
$ go get github.com/gorilla/mux
$ go get -u gopkg.in/resty.v1
```
- 6.Crie ```http-rest-client.go``` onde definiremos manipuladores que chamam manipuladores resty, como ```GET, POST, PUT e DELETE```, obtêm a resposta do serviço REST e gravam em um fluxo de resposta HTTP

```
package main
import 
(
  "encoding/json"
  "fmt"
  "log"
  "net/http"
  "github.com/gorilla/mux"
  resty "gopkg.in/resty.v1"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8090"
)
const WEB_SERVICE_HOST string = "http://localhost:8080"
type Employee struct 
{
  Id string `json:"id"`
  FirstName string `json:"firstName"`
  LastName string `json:"lastName"`
}
func getEmployees(w http.ResponseWriter, r *http.Request) 
{
  response, err := resty.R().Get(WEB_SERVICE_HOST + 
  "/employees")
  if err != nil 
  {
    log.Print("error getting data from the web service :: ", err)
    return
  }
  printOutput(response, err)
  fmt.Fprintf(w, response.String())
}
func addEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  decodingErr := json.NewDecoder(r.Body).Decode(&employee)
  if decodingErr != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", decodingErr)
    return
  }
  log.Printf("adding employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  response, err := resty.R().
  SetHeader("Content-Type", "application/json").
  SetBody(Employee{Id: employee.Id, FirstName: 
  employee.FirstName, LastName: employee.LastName}).
  Post(WEB_SERVICE_HOST + "/employee/add")
  if err != nil 
  {
    log.Print("error occurred while adding employee :: ", err)
    return
  }
  printOutput(response, err)
  fmt.Fprintf(w, response.String())
}
func updateEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  decodingErr := json.NewDecoder(r.Body).Decode(&employee)
  if decodingErr != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", decodingErr)
    return
  }
  log.Printf("updating employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  response, err := resty.R().
  SetBody(Employee{Id: employee.Id, FirstName: 
  employee.FirstName, LastName: employee.LastName}).
  Put(WEB_SERVICE_HOST + "/employee/update")
  if err != nil 
  {
    log.Print("error occurred while updating employee :: ", err)
    return
  }
  printOutput(response, err)
  fmt.Fprintf(w, response.String())
}
func deleteEmployee(w http.ResponseWriter, r *http.Request) 
{
  employee := Employee{}
  decodingErr := json.NewDecoder(r.Body).Decode(&employee)
  if decodingErr != nil 
  {
    log.Print("error occurred while decoding employee 
    data :: ", decodingErr)
    return
  }
  log.Printf("deleting employee id :: %s with firstName 
  as :: %s and lastName as :: %s ", employee.Id, 
  employee.FirstName, employee.LastName)
  response, err := resty.R().
  SetBody(Employee{Id: employee.Id, FirstName: 
  employee.FirstName, LastName: employee.LastName}).
  Delete(WEB_SERVICE_HOST + "/employee/delete")
  if err != nil 
  {
    log.Print("error occurred while deleting employee :: ", err)
    return
  }
  printOutput(response, err)
  fmt.Fprintf(w, response.String())
}
func printOutput(resp *resty.Response, err error) 
{
  log.Println(resp, err)
}
func main() 
{
  router := mux.NewRouter().StrictSlash(false)
  router.HandleFunc("/employees", getEmployees).Methods("GET")
  employee := router.PathPrefix("/employee").Subrouter()
  employee.HandleFunc("/add", addEmployee).Methods("POST")
  employee.HandleFunc("/update", updateEmployee).Methods("PUT")
  employee.HandleFunc("/delete", deleteEmployee).Methods("DELETE")
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 7.Rode no terminal

```$ go run http-rest-client.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8090

- A execução de uma solicitação GET para o cliente REST na linha de comando da seguinte maneira fornecerá uma lista de todos os funcionários do serviço

```$ curl -X GET http://localhost:8090/employees```

```[{"id":"1","firstName":"Foo","lastName":"Bar"},{"id":"2","firstName":"Baz","lastName":"Qux"}] ```

- Execute http-rest-post.go, que criamos em uma de nossas receitas anteriores, em um terminal separado executando o seguinte comando

```$ go run http-rest-post.go```

- Executar uma solicitação POST para o cliente REST na linha de comando

```$ curl -H "Content-Type: application/json" -X POST -d '{"Id":"3", "firstName":"Quux", "lastName":"Corge"}' http://localhost:8090/employee/add```
```[{"id":"1","firstName":"Foo","lastName":"Bar"},{"id":"2","firstName":"Baz","lastName":"Qux"},{"id":"3","firstName":"Quux","lastName":"Corge"}]```

- Isso adicionará um funcionário à lista estática inicial e retornará uma lista atualizada dos funcionários

**Entendendo as partes**
- Usando ```import ("codificação / json" "fmt" "log" "net / http" "github.com/gorilla/mux" resty "gopkg.in/resty.v1")```, nós importamos ```github.com/gorilla/mux``` para crie um Gorilla Mux Router e ```gopkg.in/resty.v1``` com o alias do pacote como resty, que é um cliente REST do Go, tendo vários manipuladores para consumir o serviço da web RESTful

- Usando c```onst WEB_SERVICE_HOST string = "http: // localhost: 8080"```, declaramos a URL completa do host do serviço da web RESTful
>Dependendo do tamanho do projeto, você pode mover a cadeia WEB_SERVICE_HOST para o arquivo de constantes ou para o arquivo de propriedades, ajudando-o a substituir seu valor no tempo de execução

- Definimos um manipulador ```getEmployees``` onde criamos um novo objeto de solicitação resty chamando seu manipulador```R()```, chamamos o método ```Get```, que executa a solicitação HTTP GET, obtém a resposta e a grava em uma resposta HTTP

- Definimos mais três manipuladores que fazem as solicitações ```POST, PUT e DELETE``` para o serviço RESTful e um ```main ()``` onde criamos uma instância do roteador ```gorilla / mux``` e registramos o caminho do URL ```/ employees``` com o manipulador ```getEmployees``` e ```/ employee / add, / employee / update``` e ```/ employee / delete``` com os manipuladores ```addEmployee```, ```updateEmployee``` e ```deleteEmployee```
