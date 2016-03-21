---
title: Processamento em lote

language_tabs:
  - xml: XML

toc_footers:
  - <a href='/Webservice-1.5/'>Integração Webservice 1.5</a>
  - <a href='/Webservice-1.5-FAQ'>Perguntas frequentes</a>

search: true
---

# Batch Processing

The Batch Processing allows to transmit a set with multiple transactions in a single call, these transactions will be processed and made available through a return file in XML format.

## What is the Batch Processing?

The Batch Processing is based on an XML file, which must contain a list of transactions that make up the lot, these transactions should not require authentication, it will be executed directly by Cielo, after generating the file with the Commercial Establishment, should be sent to Cielo using the HTTPS protocol.

Upon receipt of the file, a pre-validation is done by Cielo and it goes into a queue for processing, and this process is scheduled to run hourly.

At the time of pre-validation file may be rejected if it is sent blank or invalid formatting.

After a period of two hours the Commercial Establishment may request the download of the return file.

# Integration

## Rules

Batch Processing should follow these rules:

<aside class="warning">The lot must respect the limits provided below. If not respected, the file will be rejected.</aside>

1. The XML for batch processing is set by ecm-lote.xsd that has dependence on the ecommerce.xsd.
2. The file must respect the 20MB limit (about: 27,000 transactions).
3. Batch processing supports the same types of online transactional operations. See table attached.
   - The Commercial Establishment data will be informed once inside the lot, i.e., the transaction packet belongs exclusively to the Establishment informed on file.
   - The lot to be generated must contain one or more transactions of the same type of operation, if more than one type is sent in the batch, the file will be rejected.
4. The naming of the file must respect the following rule and contain the following structure and field sizes:
  - Example: `ECOMM_1006993069_02_20121002_0000000086.xml`
       - a) Prefix **- shall start with "ECOMM";
       - b) Membership EC** - Number of affiliation Commercial Establishment with Cielo, will contain ten digits;
       - c) Type of operation** - must contain two digits and the code for the type of operation see table above;
       - d) Date of file generation** - the date on which the file was generated, it must be in yyyymmdd format;
       - e) Batch number** - lot number should be sequential and contain ten digits padded with leading zeros;
       - f) File extension** - XML should be mandatory.

## Messages
 
### FILE UPLOAD MESSAGE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<retorno-upload-lote xmlns="http://ecommerce.cbmp.com.br ">
    <data-envio>2012-10-08T09:38:04.284-03:00</data-envio>
    <data-retorno>2012-10-09T09:38:04.284-03:00</data-retorno>
    <mensagem>Seu lote está válido para processamento. Favor aguardar o determinado para retorno.</mensagem>
</retorno-upload-lote>
```
After the file generation, the Commercial Establishment must make your upload via the HTTPS protocol, using the POST method, at the following URL [http://ecommerce.cbmp.com.br/lote/ecommwsecLoteUpload.do](http://ecommerce.cbmp.com.br/lote/ecommwsecLoteUpload.do).

After sending, the Commercial Establishment will receive the following XML return:

### RETURN DOWNLOAD REQUEST MESSAGE

The Commercial Establishment may request to download the file return, after two hours at least, riding the following message:

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

The table below details the XML tags that can be sent in the message to set the transaction settings for the Batch Processing:

|ELEMENT|KIND|MANDATORY|SIZE|DESCRIPTION|
|--------|----|---------------|-------|---------|
|Data-sending||n/a|n/a|n/a|Date and send time of the file upload message from the Commercial Establishment|
|Date Return|n/a|n/a|n/a|Date and time that Cielo responded to the Commercial Establishment receipt of the filen/a|n/a|
|Messagen/a|n/a|Reply message sent by Cielo|
|Number-lot|n/a|n/a|n/a|1..10	Batch number which was requested uploading|

Upon receiving this message, Cielo e-commerce platform will check if the batch is processed and the file is generated in the outbox, with positive validations, return file is returned to the Commercial Establishment, otherwise an XML will be returned whose message reports on what stage the case is pending.

If the file has been generated, however not in the outbox, may have been cleaning the storage, in this case, automatically, an event occurs that calls for the duplicate file. The Commercial Establishment will be informed via an XML return message; a new request shall be made later, so that a new file is generated.

# Attachment

## Types of online transactional operations

|TYPE OF OPERATION|CODE|
|Authorization|02|
|Cancellation|03|
|Catch|04|
|Tokenization|05|
|Query|06|
|Query ChSeq|07|
|Autorization Tid|08|

## Response Codes

The errors that may appear in the XML message by TAG are arranged as follows:

|CODE|ERROR|DESCRIPTION|ACTION|
|------|----|---------|----|
|071|Inconsistency in the file format	Invalid batch file is not an XML with invalid format	Review the file formatting|
|072|This file has been submitted for processing	Duplicate lot, there is already a lot with the same number for EC	Review the numerical sequence of lots|
|073|Invalid type of transaction	More than one type of operation in batch	Review the types of transactions that are included in the batch|
|074|No existent file	File is not in the Cielo ecommerce platform	Review file information previously sent|
|075|Invalid XML formatting	Parse error file, xml formatting in invalid file	Review the formatting of XML|
|076|Incorrect naming	Incorrect file naming	Review file name structure|
|079|Unexpected error	System failure	Persists, contact Support.|
|080|Inconsistency in the content and nomenclature	Different types of present operations on the file content and naming	Reviewing content and nomenclature of the file|
|081|Batch processing	Batch not processed yet	Send new request later|
|082|Expired file	Batch file expired	Send new request later|
|083|Invalid batch number	Invalid batch number	Reviewing lot number|
|084|Invalid number of EC	Invalid number of EC	Review the EC number|
|085|Invalid Credentials	Invalid Credentials	Reviewing credentials|
