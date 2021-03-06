# Trabalho especificando componentes equipe 05
## INF331 2020 Componentização e Reúso de Software
### Instituto de Computação Universidade Estadual de Campinas

#### Equipe 05 - Alunos participantes
- Alexsandro Coltre Gusmão		 32.538.244-X
- Felipe Villas Boas 			43.574.474-4
- Júlio César Martins dos Anjos Silva 	34.394.879-5
- Lucas Oliveira dos Santos 		37.870.978-1
- Marcos Aurelio Fabricio 		 MG-8.148.986
- William Roberto dos Santos Pereira   43.681.396-8

Baseado no tema “Gerenciamento de Fornecedores em MarketPlace”, elaboramos uma modelagem baseadas em componentes, pensando nas principais funcionalidades do sistema, e em uma mecânica de vendas como um "leilão invertido". Na etapa dois, detalhamos subcomponentes da etapa de compra.

## Diagrama Nível 1
![DiagramaNivel1](https://github.com/acgusmao/inf331Equipe05TrabalhoFinal/blob/master/images/DiagramaNivel1.png?raw=true)

## Descrição dos processos

O MarketPlace descrito pela equipe possui duas formas de se realizar uma compra:
Convencional: O cliente escolhe um produto, depois um fornecedor baseado numa lista com os preços que cada um oferece pelo produto em questão e o adiciona no carrinho.
* Leilão invertido: O cliente escolhe um produto, clica em Realizar Leilão e então o processo de leilão invertido é iniciado. Após a finalização processo os 3 fornecedores com melhores preços são informados ao cliente que pode escolher qual deseja inserir no carrinho.
* Sendo assim, descrevemos em nível de componentes o processo de leilão, desde o trigger de início até a escolha do fornecedor com melhor preço, e o processo de compra convencional, desde a busca do produto até o final da transação financeira.

## Descrição do Processo de Leilão

* O componente Formulário Início Leilão assina uma mensagem com o tópico View/Leilao/{produtoId} através da interface IProduto e posta uma mensagem com o tópico Control/Leilao/Inicio/{produtoId} com a interface IPedido, informando assim ao barramento sobre o início do leilão.
* O componente Leilão assina uma mensagem com o tópico Control/Leilao/Inicio/{produtoId}, organiza as informações sobre o leilão e posta uma mensagem com o tópico Control/Leilao/Produto/{id} através da interface IProdutoLeilao.
* O componente FornecedorCRUD que assina o tópico Control/Leilao/Produto/{id} através da interface IProdutoLeilao recebe os dados do produto que está no barramento no processo de leilão. Cada fornecedor que assina a mensagem do produto em questão do barramento receberá a mensagem e terá a chance de postar seu preço. Os dados do fornecedor com o devido preço do produto são enviados ao barramento pela interface IProdutoLeilao com o tópico Leilao/Produto/Fornecedor/Valor.
* O componente Seleção Fornecedores assina o tópico Leilao/Produto/Fornecedor/Valor através da interface IFornecedor. Após receber um número suficiente de mensagens, o componente e lista os 3 fornecedores com menor preço e posta a lista de fornecedores no barramento através da interface Lista Fornecedores com o tópico Leilao/Lista/Produto/fornecedor1+fornecedor2+fornecedor3.
* Finalizando o processo de leilão o componente Formulário Escolha Fornecedor assina a mensagem postada pelo componente descrito anteriormente e então as exibe em tela para que o cliente possa escolher o fornecedor com o qual realizará a compra. Após a escolha, os dados são enviados para o barramento através da interface IProduto com o tópico View/Cart/{produtoId}.

## Descrição do Processo de Compra
* O componente Formulário Buscar Produtos assina uma mensagem com o tópico View/Buscar/{produtoId} através da interface Lista de Produtos e posta uma mensagem no barramento com o tópico View/Get/{produtoId} pela interface IProduto.
* O componente Formulario Produto recebe o produto no barramento através da interface IProduto com a assinatura View/Get/{produtoId} e após o cliente escolher comprá-lo, é disparada uma mensagem com o tópico View/Cart/{produtoId} através da interface IProduto.
* Então o componente Formulário Carrinho assina uma mensagem com o tópico View/Cart/{produtoId} através da interface IProduto. Esse processo pode ser realizado inúmeras vezes, para que o carrinho seja montado com n produtos. Após a montagem do carrinho, é postado no barramento uma mensagem com o tópico Control/Compra/Cart/Finalizado através da interface ICart.
* O componente Comprar Produto recebe um carrinho já montado, com todas as informações dos produtos, bem como do cliente que está realizando a compra através da interface ICart pela mensagem Control/Compra/Cart/Finalizado. Então toda a compra é organizada dentro desse componente, preenchendo dados como método de entrega e método de envio que seguem para para o barramento pela interface ITransaction com a assinatura Control/Compra/Transaction/{id}.
* Então o componente Transações FInanceiras Seguras que assina a mensagem Control/Compra/Transaction/{id} através da interface ITransaction, recebe todos os dados necessários para criptografar os dados referentes à transaction. Esse componente se fez necessário pois a segurança da operação financeira é um requisito do nosso MarketPlace. Então, os dados criptografados são postados no barramento através da interface IDataSafe com a assinatura Control/TransactionSafe/{id}.
* Por fim, o componente Protocolo de Privacidade recebe essa transação financeira criptograda através da interface IDataSafe e realiza a comunição com a empresa responsável pelo pagamento, se encarregando de finalizar as operações financeiras de toda a operação de compra. Após a finalização da operação, é gerada uma string com as informações relevantes da transação que é postada no barramento com o tópico Control/Transaction/{id]/Finished pela interface IData.

## Diagrama Nível 2
![DiagramaNivel2](https://github.com/acgusmao/inf331Equipe05TrabalhoFinal/blob/master/images/DiagramaNivel2.png?raw=true)

## Descrição do Componente Comprar Produto

* O componente Comprar Produto assina no barramento mensagens de tópico “Control/Compra/Cart/Finalizado” através da interface ICart.
* A interface ICart se encarrega de entregar ao componente Comprar Produto um carrinho de compras contendo uma lista de produtos, bem como os dados do usuário do marketplace que está efetuando a compra.
* Dentro do componente Comprar Produto os dados chegam até o subcomponente GerenciaCompra que é o responsável por organizar a comunicação entre os outros subcomponentes
* Os dados que vieram do subcomponente Escolha Método de Envio dentro do componente Formulário Compra (View) são enviados juntamente com os dados do carrinho fornecidos pela ICart até o subcomponente Método de Envio que os organizará para que sejam parte da transação financeira que será formada nesse componente.
* De posse do método de envio, o subcomponente Taxa de Entrega, vai através da interface Gerenciar Taxa de Entrega organizar as informações já existentes a respeito desse tópico, bem como preparar a mensagem para cálculo do frete.
* A interface ConsultaTaxaPorCep envia uma mensagem para a API dos Correios a fim de obter subsídios para o cálculo do frete para envio do carrinho recém montado para o endereço cadastrado pelo cliente.
* Também é verificado se os produtos constantes no carrinho ainda se encontram disponíveis com o fornecedor escolhido. Para tal, é utilizado o componente ConsultaDeEstoque que se comunica com o componente GerenciaCompra através da interface Verifica Estoque.
* Como essa consulta necessita ser feita diretamente com o fornecedor de cada um produtos, a interface Consulta Estoque do Lojista posta uma mensagem com a assinatura estoque/{idFornecedor}/{idProduto} através da interface Solicita Estoque do Lojista e assina uma mensagem com a assinatura estoque/{idFornecedor}/{idProduto}/status através da interface Recebe Estoque do Lojista.
* Para a realização do pagamento o componente GerenciaCompra utiliza os dados provenientes do subcomponente Escolha Método de Pagamento dentro da View Formulário Compras e  os  envia para o subcomponente  FormaDePagamento para prosseguir com a montagem da transação.
* Caso o método escolhido de pagamento seja cartão de crédito, o componente envia uma mensagem através da interface Consulta a Administradora do Cartão para saber se a transação pode prosseguir.
* De posse de todos os dados necessários, o subcomponente GerenciaCompra os envia para o MontaTransacao através da interface Envia Dados Compra que os prepararão para serem recebidos pelo macro componente responsável pela segurança dos dados.
* Neste ponto existem dois caminhos concomitantes:
	- Os dados da compra são enviados para o subcomponente Salva Dados Compra dentro do Model Compra através da inferface Dados Compra. Esse componente dentro da Model é responsável por persistir todos os dados da transação no banco de dados
	- Os dados da compra são enviados para o componente Transações Financeiras Seguras através da interface ITransaction onde o processo de compra dos produtos terá prosseguimento.

## Descrição dos componentes

* Componente FormulárioCadastroCliente – Componente da view onde é possível o usuário o acesso para operações de create, retrieve, update e delete para clientes. Assinaturas deste componente no barramento:
<br/>View\Cliente\get\{id}
<br/>View\Cliente\post\{id}
<br/>View\Cliente\put\{id}
<br/>View\Cliente\delete\{id}
<br/>Interface provida : ICliente
<br/>Interface requerida: ICliente
 
* Componente FormulárioCadastroFornecedor – Componente da view onde é possível o usuário o acesso para operações de create, retrieve, update e delete para fornecedores. Assinaturas deste componente no barramento:
<br/>View\Fornecedor\get\{id}
<br/>View\Fornecedor \post\{id}
<br/>View\Fornecedor \put\{id}
<br/>View\Fornecedor \delete\{id}
<br/>Interface provida : IFornecedor
<br/>Interface requerida: IFornecedor
  
* Componente FormulárioCadastroProdutos – Componente da view onde é possível o usuário o acesso para operações de create, retrieve, update e delete para fornecedores. Assinaturas deste componente no barramento:
<br/>View\Produto\get\{id}
<br/>View\Produto\post\{id}
<br/>View\Produto\put\{id}
<br/>View\Produto\delete\{id}
<br/>Interface provida : IProduto
<br/>Interface requerida: IProduto
 
* Componente FormulárioHistóricoVendas – Componente da view onde é possível o fornecedor o acesso ao histórico de vendas. Assinaturas deste componente no barramento:
<br/>View\Fornecedor\{id}\historicoVendas\Lista
<br/>Interface provida : Lista
<br/>Interface requerida: IFornecedor
 
* Componente FormulárioProduto – Componente da view para visualizar um determinado produto e adicionar no carrinho. Assinaturas deste componente no barramento:
<br/>View\Produto\post\{id}\Cart
<br/>View\Produto\delete\{id}\Cart
<br/>Interface provida : IProduto
<br/>Interface requerida: IProduto
 
*  Componente FormulárioCarrinho – Componente da view para visualizar o carrinho de compra, adicionar ou remover produtos. Assinaturas deste componente no barramento:
<br/>View\Cliente\{id}\Cart\Comprar
<br/>Interface provida : ICart
<br/>Interface requerida: IProduto
 
* Componente FormuláriodeLogin – Componente da view para realizar a autenticação dos usuarios. Assinaturas deste componente no barramento:
<br/>View\Login\User\{id}
<br/>Interface provida : IKey
<br/>Interface requerida: IUser
 
* Componente FormulárioInicioLeilão – Componente da view para realizar o inicio do leilão invertido. Assinaturas deste componente no barramento:
<br/>View\Leilao\Produto\{id}\Pedido
<br/>Interface provida : IPedido
<br/>Interface requerida: IProduto
 
* Componente Leilão – Componente de controle para realizar os procedimentos de realização do leilão invertido. Assinaturas deste componente no barramento:
<br/>View\Leilao\Produto\{id}\Lance
<br/>Interface provida : IProdutoLeilao
<br/>Interface requerida: IPedido
 
* Componente SeleçãodosFornecedores – Componente de controle para selecionar o menores lances dos fornecedores do leilão invertido. Assinaturas deste componente no barramento:
<br/>View\Leilao\Produto\{id}\ListaDeFornecedores
<br/>Leilao/Lista/Produto/fornecedor1+fornecedor2+fornecedor3
<br/>Interface provida : ListaFornedores
<br/>Interface requerida: IFornecedor
 
* Componente FormulárioBuscarProdutos – Componente da view que realiza a busca de produtos. Assinaturas deste componente no barramento:
<br/>View/Buscar/Produto/
<br/>Interface provida : ListaProdutos
<br/>Interface requerida: IProduto
 
* Componente FornecedorCRUD – Componente de controle onde é realizada as operações de create, retrieve, update e delete para fornecedores. Assinaturas deste componente no barramento:
<br/>Control\ Fornecedor \post\{id}
<br/>Control\ Fornecedor \put\{id}
<br/>Control\ Fornecedor \delete\{id}
<br/>Control\Fornecedor\post\{id}\Leilao\{LeilaoId}\Produto\{ProdutoId}
<br/>Control\Fornecedor\get\{id}
<br/>Interface provida : ICliente
<br/>Interface requerida: ICliente
<br/>Interface provida : IProdutoLeilao
<br/>Interface requerida: IProdutoLeilao
 
* Componente ClienteCRUD – Componente de controle onde é realizada as operações de create, retrieve, update e delete para clientes. Assinaturas deste componente no barramento:
<br/>Control\Cliente\get\{id}
<br/>Control\Cliente\post\{id}
<br/>Control\Cliente\put\{id}
<br/>Control\Cliente\delete\{id}
<br/>Interface provida : ICliente
<br/>Interface requerida: ICliente
 
* Componente ProdutoCRUD – Componente de controle onde é realizada as operações de create, retrieve, update e delete para produtos. Assinaturas deste componente no barramento:
<br/>Control\Produto\get\{id}
<br/>Control\Produto \post\{id}
<br/>Control\Produto \put\{id}
<br/>Control\Produto \delete\{id}
<br/>Interface provida : IProduto
<br/>Interface requerida: IProduto
 
* Componente HistóricodeVendas – Componente de controle que realiza a busca do histórico de vendas para um determinando fornecedor. Assinaturas deste componente no barramento:
<br/>Control \Historico\Vendas\Fornecedor\{id}
<br/>Interface provida : Lista
<br/>Interface requerida: IFornecedor

* Componente FormulárioEscolhaFornecedor – View onde o cliente escolhe um fornecedor entre os retornados pelo leila e adiciona o produto no carrinho. Assinaturas desta componente no barramento:
<br/>Leilao/Lista/Produto/fornecedor1+fornecedor2+fornecedor3
<br/>View/Cart/{produtoId}
<br/>Interface provida : IProduto
<br/>Interface requerida: LIstaFornecedores

 
* Componente HistóricodeOperações – Componente de controle que realiza a busca do histórico de operações para um determinando usuario. Assinaturas deste componente no barramento:
<br/>Control \Historico\Operacoes\Fornecedor\{id}
<br/>Interface provida : Lista
<br/>Interface requerida: IFornecedor
 
* Componente ComprarProduto – Componente de controle que realiza o processo de compra para um determinada produto. Assinaturas deste componente no barramento:
<br/>Control \Compra\Cart\Finalizado
<br/>Interface provida : ITransaction
<br/>Interface requerida: ICart
 
* Componente RecomendarFornecedor – Componente de controle que realiza o processo de recomendação do fornecedor. Assinaturas deste componente no barramento:
<br/>Sem Assinatura no barramento
<br/>Interface provida :
<br/>Interface requerida: IFornecedor
 
* Componente LogindeUsuários – Componente de controle que realiza o processo de autenticação dos usuarios. Assinaturas deste componente no barramento:
<br/>Control \Login\Usuario\{id}
<br/>Interface provida : Ikey
<br/>Interface requerida: IUser
 
* Componente ProtocolodePrivacidade - Componente de controle que realiza o processo de criptografia dos dados pessoais. Assinaturas deste componente no barramento:
<br/>Control \Privacidade\Data
<br/>Interface provida : IDataSafe
<br/>Interface requerida: IData
 
* Componente TransaçõesFinanceirasSeguras - Componente de controle que realiza o processo de segurança nas operações financeiras. Assinaturas deste componente no barramento:
<br/>Control \Safe\Data
<br/>Interface provida : IDataSafe
<br/>Interface requerida: ITransaction

## Diagrama de Classes das Interfaces

![DiagramaDeClassesInterfaces](https://github.com/acgusmao/inf331Equipe05TrabalhoFinal/blob/master/images/DiagramaClasseDasInterface.png?raw=true)

## Estrutura JSON Interfaces

~~~json
"ICliente":
{
	"cpf": "233.548.144-21",
	"ICrud": {
		"command": 1,
		"operation": 3,
		"status": 0
	},
	"IUser": {
		"name": "Pedro Henrique", 
		"adrress": "Rua Alvorada",
		"fone": "(11)9999-2164",
		"userType": 2,
		"ILogin": {
			"username": "Pedro123",
			"password": "********"
		}
	},
	"IData": 
	{
		"objectID": 652345324564356,
		"data": "data"
	}	
}

"IFornecedor":
{
	"cnpj": "1234.123345.124445.1",	
	"ICrud": {
		"command": 1,
		"operation": 3,
		"status": 0
	},
	"IUser": {
		"name": "Pedro Henrique", 
		"adrress": "Rua Alvorada",
		"fone": "(11)9999-2164",
		"userType": 2,
		"ILogin": {
			"username": "Pedro123",
			"password": "********"
		}
	},
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	},
	"IProdutoLeilao": {
		"fornecedor": "IFornecedor",
		"value": 15000.00,
		"IPedido": {
			"cliente": "ICliente",
			"produto": "IProduto",
			"leilaoId": 1,
			"IData": {
				"objectID": 652345324564356,
				"data": "data"
			}
		}
	}	
}

"IProduto": 
{
	"name": "Smart TV LG 42",
	"description": "Tela Led",
	"price": 2300.00,
	"quantity": 200,
	"ICrud": {
		"command": 1,
		"operation": 3,
		"status": 0
	},
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	}
}


"ICart": 
{
	"prodList": ["Produto 1", "Produto 2", "Produto 3", "Produto 4",  "Produto 5"],
	"cliente": "ICliente",
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	}
}

"IKey":
{
	"acess": 1,
	"user": "IUser",
	"keypass": 5679799090,	
	"IDataSafe": {
		"cryptoType": 345543,
		"key": 1000001,
		"IData": {
			"objectID": 652345324564356,
			"data": "data"
		}
	}
}

"IDataSafe": 
{
	"cryptoType": 345543,
	"key": 1000001,
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	}
}

"ITransaction":
{
	"cartList": ["cart 1", "cart 2", "cart 3", "cart 4",  "cart 5"],
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	}
}

"IPedido": 
{
	"cliente": "ICliente",
	"produto": "IProduto",
	"leilaoId": 1,
	"IData": {
		"objectID": 652345324564356,
		"data": "data"
	}
}
~~~

## Estratégia Multi-Plataforma

O padrão de projeto adotado para resolver o problema de implementação para criação de interfaces gráficas multi-plataformas, segue conforme o design Pattern Abstract Factory.
Como estratégias todos os componentes visuais implementam a interface ViewForm e cada fábrica concreta tem a resposabilidade de criar os objetos respectivos de cada plataforma.
O diagrama de classe abaixo representa a estratégia para a solução multi-plataforma para o marketplace.

![DiagramaDeClassesEstrategia](https://github.com/acgusmao/inf331Equipe05TrabalhoFinal/blob/master/images/AbstractFactory.png?raw=true)

