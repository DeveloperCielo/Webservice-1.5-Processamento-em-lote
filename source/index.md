---
title: Processamento em lote

language_tabs:
  - xml: XML

toc_footers:
  - <a href='/Webservice-1.5/'>Integração Webservice 1.5</a>
  - <a href='/Webservice-1.5-FAQ'>Perguntas frequentes</a>

search: true
---

# Processamento em lote

O Processamento em Lote permite que sejam transmitidas em uma única chamada um conjunto
com várias transações, essas transações serão processadas e disponibilizadas através de um arquivo de
retorno no formato XML.

## O que é o Processamento em Lote

O Processamento em Lote é baseado em um arquivo XML, o qual deve conter uma lista com as transações que compõem o lote, estas transações não devem exigir autenticação, pois será executada diretamente pela Cielo, após a geração do arquivo pelo Estabelecimento Comercial, deve ser enviado para Cielo utilizando o protocolo HTTPS.


Após o recebimento do arquivo, é feita uma pré-validação pela Cielo e o mesmo entra em uma fila para processamento, sendo que este processo é agendado para ser executado de hora em hora.

No momento da pré-validação arquivo poderá ser rejeitado, caso seja enviado em branco ou com formatação inválida.

<aside class="notice">Após o prazo de doze horas o Estabelecimento Comercial poderá solicitar o download do arquivo de retorno.</aside>

# Integração

## Regras

Processamento em Lote deverá seguir as seguintes regras:

<aside class="warning">O lote deverá respeitar os limites informados abaixo. Caso não sejam respeitados, o arquivo será rejeitado.</aside>

1. O XML para processamento em lote está definido através do ecm-lote.xsd que possui dependência com o ecommerce.xsd.
2. O arquivo deverá respeitar o limite de 20MB (aprox.: 27.000 transações).
3. O processamento em lote suporta os mesmos tipos de operações do transacional online. Veja a tabela em anexo.
   - Os dados do Estabelecimento Comercial serão informados uma única vez dentro do lote, ou seja, o pacote de transações pertence exclusivamente ao Estabelecimento informado no arquivo.
   - O lote a ser gerado deverá conter uma ou mais transações do mesmo tipo de operação, caso mais de um tipo seja enviado no lote, o arquivo será rejeitado.
4. A nomenclatura do arquivo deverá respeitar a regra abaixo e conter a seguinte estrutura e tamanhos de campo:
   - Exemplo: `ECOMM_1006993069_02_20121002_0000000086.xml`
       - a) Prefixo – obrigatoriamente deve iniciar com “ECOMM”;
       - b) N.o Afiliação EC – número de afiliação do Estabelecimento Comercial, com a Cielo, deverá conter dez dígitos;
       - c) Tipo de operação – deverá conter dois dígitos e para o código do tipo de operação vide tabela acima;
       - d) Data de geração do arquivo – data em que o arquivo foi gerado, deve ser no formato yyyymmdd;
       - e) Numero do lote – número do lote deverá ser sequencial e conter dez dígitos preenchidos com zeros à esquerda;
       - f) Extensão do arquivo – deve ser XML obrigatoriamente.

## Mensagens

### Mensagem de Upload de Arquivo

```xml
<?xml version="1.0" encoding="UTF-8"?>
<retorno-upload-lote xmlns="http://ecommerce.cbmp.com.br ">
    <data-envio>2012-10-08T09:38:04.284-03:00</data-envio>
    <data-retorno>2012-10-09T09:38:04.284-03:00</data-retorno>
    <mensagem>Seu lote está válido para processamento. Favor aguardar o determinado para retorno.</mensagem>
</retorno-upload-lote>
```

Após a geração do arquivo, o Estabelecimento Comercial deverá efetuar o seu upload através do protocolo HTTPS, utilizando o método POST, na seguinte URL [http://ecommerce.cbmp.com.br/lote/ecommwsecLoteUpload.do](http://ecommerce.cbmp.com.br/lote/ecommwsecLoteUpload.do).

Após o envio, o Estabelecimento Comercial receberá o seguinte XML de retorno:

### Mensagem de Solicitação de Download de Retorno

```xml
<?xml version="1.0" encoding="UTF-8"?>
<requisicao-download-retorno-lote versao="Versao da msg" id=“session id”>
   <dados-ec>
   <numero>1006993069 </numero>
   <chave>25fbb997438630f30b112d033ce2e621b34f3</chave>
   </dados-ec>
   <numero-lote>0000000086</numero-lote>
</requisicao-download-retorno-lote>
```


O Estabelecimento Comercial poderá solicitar o download do arquivo de retorno, após doze horas no mínimo, montando a seguinte mensagem:

A tabela abaixo detalha as TAGS do XML que podem ser enviadas na mensagem para definir as configurações da transação para o Processamento em Lote:

|Elemento|Tipo|Obrigatoriedade|Tamanho|Descrição|
|--------|----|---------------|-------|---------|
|data-envio|n/a|n/a|n/a|Data e horário de envio da mensagem de upload de arquivo pelo Estabelecimento Comercial|
|data-retorno|n/a|n/a|n/a|Data e horário que a Cielo respondeu para o Estabelecimento Comercial o recebimento do arquivo|
|mensagem|n/a|n/a|n/a|Mensagem de resposta enviado pela Cielo|
|numero-lote|N|Sim|1..10|Número do lote que foi solicitado o upload|

Ao receber esta mensagem, a plataforma Cielo eCommerce verificará se o lote está processado e o se o arquivo está gerado no outbox, com as validações positivas, o arquivo de retorno é devolvido para o Estabelecimento Comercial, caso contrário, será retornado um XML cuja mensagem informa em qual etapa o processo está pendente.

Caso o arquivo tenha sido gerado, porem não está no outbox, pode ter ocorrido limpeza do storage, neste caso, automaticamente, ocorre um evento que solicita a segunda via do arquivo. O Estabelecimento Comercial será informado através de uma mensagem XML retorno, que uma nova requisição deverá ser feita mais tarde, para que um novo arquivo seja gerado.

# Anexo

## Tipos de operações do transacional online

| Tipo de operação | Código |
|------------------|--------|
|Autorização|02|
|Cancelamento|03|
|Captura|04|
|Tokenização|05|
|Consulta|06|
|ConsultaChSeq|07|
|AutorizaçãoTid|08|

## Códigos de resposta

Os erros que podem ser apresentados na mensagem XML, através da TAG <erro>, estão dispostos a seguir:

|Código|Erro|Descrição|Ação|
|------|----|---------|----|
|071|Inconsistência no formato do arquivo|Lote inválido, arquivo não é um XML com formato inválido|Rever a formatação do arquivo|
|072|Este arquivo já foi enviado para processamento|Lote duplicado, já existe um lote com o mesmo numero para o EC|Rever a sequência numérica dos lotes|
|073|Tipo de transação inválido|Mais de um único tipo de operação no lote|Rever os tipos de transações que estão contemplados no lote|
|074|Arquivo inexistente|Arquivo não consta na plataforma Cielo eCommerce|Rever informações do arquivo enviado anteriormente|
|075|Formatação do XML inválida|Erro de parse do arquivo, formatação do xml no arquivo inválida|Rever a formatação do xml|
|076|Nomenclatura incorreta|Nomenclatura do arquivo incorreta|Rever estrutura do nome do arquivo|
|079|Erro inesperado|Falha no Sistema|Persistindo, entrar em contato com o Suporte.|
|080|Inconsistência no conteúdo e nomenclatura|Tipos diferentes de operações presentes no conteúdo arquivo e na nomenclatura|Rever conteúdo e nomenclatura do arquivo|
|081|Lote em processamento|Lote ainda não processado|Enviar nova requisição mais tarde|
|082|Arquivo expirado|Arquivo de lote expirado|Enviar nova requisição mais tarde|
|083|Numero de lote inválido|Numero de lote inválido|Rever número de lote|
|084|Número de EC Inválido|Número de EC Inválido|Rever número do EC|
|085|Credenciais inválidas|Credenciais inválidas|Rever credenciais|
