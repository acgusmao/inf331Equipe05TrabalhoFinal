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