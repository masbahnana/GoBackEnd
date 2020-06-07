# WebSocket
O WebSocket fornece uma conexão bidirecional, de single-socket e full-duplex entre o servidor e o cliente, tornando a comunicação em tempo real muito mais eficiente do que outras maneiras, como eventos longos de sondagem e enviados pelo servidor
Com o WebSocket, o cliente e o servidor podem falar independentemente, cada um capaz de enviar e receber informações ao mesmo tempo após o handshake inicial, reutilizando a mesma conexão do cliente para o servidor e o servidor para o cliente, o que reduz o atraso e o servidor carrega muito, permitindo que os aplicativos da web executem tarefas modernas da maneira mais eficaz. O protocolo WebSocket é suportado pela maioria dos principais navegadores, incluindo o Google Chrome, o Microsoft Edge, o Internet Explorer, o Firefox, o Safari e o Opera. Portanto, não há problemas de compatibilidade

## Criando um WebSocket Server
- 1.Instale o pacote ```github.com/gorilla/websocket``` usando o comando ```go get```

```bash
$ go get github.com/gorilla/websocket
```

- 2.Crie ```websocket-server.go```, onde faremos upgrade de uma solicitação HTTP para WebSocket, leremos a mensagem JSON do cliente e a transmitiremos para todos os clientes conectados

```go
package main 
import 
(
  "log"
  "net/http"
  "github.com/gorilla/websocket"
)
var clients = make(map[*websocket.Conn]bool)
var broadcast = make(chan Message) 
var upgrader = websocket.Upgrader{}
type Message struct 
{
  Message string `json:"message"`
}
func HandleClients(w http.ResponseWriter, r *http.Request) 
{
  go broadcastMessagesToClients()
  websocket, err := upgrader.Upgrade(w, r, nil)
  if err != nil 
  {
    log.Fatal("error upgrading GET request to a 
    websocket :: ", err)
  }
  defer websocket.Close()
  clients[websocket] = true
  for 
  {
    var message Message
    err := websocket.ReadJSON(&message)
    if err != nil 
    {
      log.Printf("error occurred while reading 
      message : %v", err)
      delete(clients, websocket)
      break
    }
    broadcast <- message
  }
}
func main() 
{
  http.HandleFunc
  (
    "/", func(w http.ResponseWriter, 
    r *http.Request) 
    {
      http.ServeFile(w, r, "index.html")
    }
  )
  http.HandleFunc("/echo", HandleClients)
  err := http.ListenAndServe(":8080", nil)
  if err != nil 
  {
    log.Fatal("error starting http server :: ", err)
    return
  }
}
func broadcastMessagesToClients() 
{
  for 
  {
    message := <-broadcast
    for client := range clients 
    {
      err := client.WriteJSON(message)
      if err != nil 
      {
        log.Printf("error occurred while writing 
        message to client: %v", err)
        client.Close()
        delete(clients, client)
      }
    }
  }
}
```

- 3.Rode no terminal

```bash
$ go run websocket-server.go
```

### Como isso deve funcionar
- Assim que executarmos o programa, o servidor WebSocket começará a escutar localmente na porta ```8080```

**Entenda as partes**

- ```import ("log" "net/http" "github.com/gorilla/websocket")``` que é um comando de pré-processamento que informa ao compilador Go para incluir todos os arquivos do log, ```net/http e github.com/gorilla./websocket```

- ```ar clients = make (map [* websocket.Conn] bool)```, criamos um mapa que representa os clientes conectados a um servidor WebSocket com``` KeyType ```como um objeto de conexão ```WebSocket``` e ```ValueType``` como Boolean

- ```ar broadcast = make (chan Mensage)```, criamos um canal onde todas as mensagens recebidas são gravadas

- Definimos um manipulador ```HandleClients```, que ao receber a solicitação HTTP ```GET```, atualiza para o ```WebSocket```, registra o cliente com o socket server, lê as mensagens JSON solicitadas e as grava no canal de transmissão

- Definimos uma função Go ```broadcastMessagesToClients```, que captura as mensagens gravadas no canal de transmissão e as envia para todos os clientes atualmente conectados ao servidor WebSocket


## Debugging seu primeiro servidor WebSocket local
O Debugging de um aplicativo da Web é uma das habilidades mais importantes para um desenvolvedor aprender, pois ajuda a identificar um problema, isola a origem do problema e corrige o problema ou determina uma maneira de contorná-lo

Para essa parte precisatemos de uma IDE, vou mostrar a configuração da IDE do GOLang a **Goland** da [JetBrains](https://www.jetbrains.com/go/) 

- 1.Clique em Open Project no GoLand IDE para abrir o websocket-server.go, que escrevemos antes

- 2.Quando o projeto for aberto, clique em Editar configurações

- 3.Selecione Adicionar nova configuração clicando no sinal +

- 4.Selecione Go Build, renomeie a configuração para WebSocket Local Debug, altere Run kind para Directory e clique em Apply e OK

- 5.Coloque alguns breakpoints e clique no botão Debug


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor WebSocket será iniciado localmente no modo de depuração, escutando na porta ```8080```

- Navegar até ```http://localhost:8080``` nos mostrará a página do cliente WebSocket com uma caixa de texto e um botão Enviar

- Digite o texto e clique no botão Enviar para ver a execução do programa parar nos breakpoints que colocamos no GoLand IDE


## Debugging remoto - WebSocket server
Aprendemos como depurar um servidor WebSocket que está sendo executado localmente. Agora aprenderemos a depurá-lo se estiver sendo executado em outra máquina ou em uma máquina remota
*Etapas são parecidas com a anterior*

- 1.Adicione outra configuração clicando em Editar configurações

- 2.Clique no sinal + para adicionar nova configuração e selecione Ir remoto

- 3.Renomeie a configuração de depuração para o ```WebSocket Remote Debug```, altere o Host para ```IP``` ou ```DNS``` da máquina remota e clique em Aplicar e OK

- 4.Execute um servidor Delve sem cabeçalho no computador de destino ou remoto executando o seguinte comando `https://www.jetbrains.com/go/`

> O comando anterior iniciará um servidor de API escutando na porta 2345

- 5.Selecione WebSocket Remote Debug configuration e clique no botão Debug


### Como isso deve funcionar
- Selecione WebSocket Remote Debug configuration e clique no botão Debug e irá aparecer a mensagem ```Hello Arpit```


## Teste unitário
**Testes unitários ou desenvolvimento orientado a testes (TDD - test-driven development)** ajudam o desenvolvedor a projetar código acoplado com foco na reutilização de código. Também nos ajuda a perceber quando parar de codificar e fazer alterações rapidamente

- Vamos utilizar o WebSocker server que fizemos antes

- 1.Instale os pacotes ```github.com/gorilla/websocket``` e ```github.com/stretchr/testify/assert``` usando o comando ```go get```

```bash
$ go get github.com/gorilla/websocket
$ go get github.com/stretchr/testify/assert
```

- 2.Crie ```websocket-server_test.go``` onde criaremos um servidor de teste, conectaremos a ele usando o cliente [Gorilla](https://www.gorillatoolkit.org/) e, eventualmente, leremos e escreveremos mensagens para testar a conexão

```go
package main
import 
(
  "net/http"
  "net/http/httptest"
  "strings"
  "testing"
  "github.com/gorilla/websocket"
  "github.com/stretchr/testify/assert"
)
func TestWebSocketServer(t *testing.T) 
{
  server := httptest.NewServer(http.HandlerFunc
  (HandleClients))
  defer server.Close()
  u := "ws" + strings.TrimPrefix(server.URL, "http")
  socket, _, err := websocket.DefaultDialer.Dial(u, nil)
  if err != nil 
  {
    t.Fatalf("%v", err)
  }
  defer socket.Close()
  m := Message{Message: "hello"}
  if err := socket.WriteJSON(&m); err != nil 
  {
    t.Fatalf("%v", err)
  }
  var message Message
  err = socket.ReadJSON(&message)
  if err != nil 
  {
    t.Fatalf("%v", err)
  }
  assert.Equal(t, "hello", message.Message, "they 
  should be equal")
}
```

### Como isso deve funcionar
- Execute ```go test```

```bash
$ go test websocket-server_test.go websocket-server.go
ok  command-line-arguments 0.048s
```

- Isso nos dará a resposta **ok**, o que significa que o teste foi compilado e executado com sucesso

- Vamos ver como fica quando um teste Go falha. Altere a saída esperada na instrução assert para outra coisa. No seguinte ```Hello``` foi alterado para ```Hi```

```go
...
assert.Equal(t, "hi", message.Message, "they should be equal")
...
```

- Execute o ```go teste``` de novo

```bash
$ go test websocket-server_test.go websocket-server.go
```

- Ele nos fornecerá a resposta de falha junto com o rastreio de erro

