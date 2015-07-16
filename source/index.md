---
title: wiCode Platform Point-of-Sale API Integration

language_tabs:
  - xml
  - java
  - php

toc_footers:
  - <a href='http://wigroupinternational.com/'>wiGroup</a>
includes:
  - errors

search: true
---

# wiPlatform Overview

The wiCode platform is a mobile transactional platform designed specifically for secure mobile (and card) transacting in retail and hospitality stores. The wiCode platform allows Value Store Providers (VSP's) to integrate to a single platform to transact at multiple retailers. It also allows retailers to integrate to a single platform to accept transactions from multiple VSP's. 

The <em>Transaction Engine</em> exposes several XML webservice endpoints. This page describes the POS Provider API which is used by retailers to process transactions.

<img src="./images/platform.png" alt="some text"/>

The POS (Point-of-sale) integration into the wiCode platform allows for the following functionality suite:

<ul>
  <li>Pay in store</li>
  <li>Redeem loyalty / coupon / voucher</li>
  <li>Cash withdrawal and deposit</li>
  <li>Earn loyalty</li>
  <li>Reward trigger on purchase</li>
</ul>

We recommend that POS providers integrate into the full suite of wiGroup functionality, however for some projects the integration scope is driven by the client’s business requirements and may take on a more of a phased approach. 

# Technical Transaction Flow

wiGroup supports dual messaging architecture. Payments using the wiCode platform consists of interactions between the issuer (POS system) and the VSP. The wiCode platform serve as a guarantor of processing the resulting settlement. The wiCode platform utilizes two basic requests to process transactions:

<ul>
<li>Transaction request</li>
<li>Advice request</li>
</ul>

The <em>transaction request</em> pushes a transaction (payment, deposit or withdrawal) to the VSP for authorization. The transaction response will contain the result of this request, which can then be applied to the transaction appropriately. 

The </em>advice request</em> should be made to the wiCode platform once the real-world transaction has been completed. This could either be a once the tender is received by the cashier or if a reversal is required. This call will either change the state of the transaction from pending to successful, or, reversed.

The typical interactions for processing a transaction is displayed in the figure below.

<img src="./images/wiGroupTEIntegraion3.png" alt="some text"/>

# Prerequisites and minimum requirements

All POS integrations should be able to handle the following transaction types:
<ul>
<li>Coupon & Voucher redemption</li>
<li>Split tender</li>
</ul>

Coupon & Vouchers requires the product objects(s) to be populated by the POS. Without a list of objects, with their respective SKU numbers, Coupons & Vouchers will not be redeemed. Coupons are associated with a SKU or list of SKUs to be associated with the basket. Once a Coupon or Voucher is processed a discount object is returned containing all the items that have been discounted.

Split tender is the process of using multiple tenders to settle a bill. In this case the total basket amount is not processed in one transaction, but rather a number of transactions. In each case the total amount processed should be less than the basket amount outstanding. The POS should have built-in logic to ensure that a bill is not closed before the total amount processed is equal to (or greater than) the basket amount. 

It is further more important for the POS to handle error codes and failed responses in a user-friendly manner. POS timeouts with regards to sending and receiving information from the wiCode Platform must be dealt with in a constructive manner. There may be numerous reasons for failure to communicate with web servers (internet connectivity to the POS is disabled or slow, a firewall may be blocking the IP address to which a web service is sent, incorrect configuration settings on the POS etc.) and the possibilities of these issues occurring should be tested during the integration phase into the wiCode Platform. 

If, for any reason, there is a break-down in communication between the POS and the wiCode Platform, and there are some transactions which were not successfully advised (the transactions were processed, but the transactions are still in an SPR state), the POS must persist to attempt send the advice call for each transaction until a successful response is returned.  

The following table contains the various transaction types and the types of wiCodes that each payment type can process.

Payment Type | Type of wiCode(s)
------------ | ---------------
PAYMENT | <li>Coupon</ul><li>Voucher</li><li>Gift Card</li><li>Mobile Wallet</li><li>Payment (linked with a user's credit card)</li><li>Loyalty points</li>
WITHDRAWAL | Mobile wallet usage
DEPOSIT | Mobile wallet top-up

# Web services

Two WSDLs are provided. The first should be used for all testing and integration processes, while the second WSDL should be configured for production and live testing in pilot stores.

In order for the Point of Sale terminal to connect to the wiCode platform to authorise transactions, any firewalls in place on the network to which the terminals connect should allow communication (request and response) with the end points detailed in this section. Please note that business and technical requirements will determine which of the end points below will be required.

## WSDL used for the development and testing phase

<aside class="notice">
The QA wiCode platform exposes SOAP web service endpoints by utilizing the following WSDL: <br>
http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl
</aside>

<aside class="success">
The destination IP address is 54.76.23.107, using either a TCP port (8080) or an SLL port connection (8181). The DNS is rad2.wigroup.co. 
</aside>

## WSDL used for production phase

<aside class="notice">
The production wiCode platform exposes SOAP web service endpoints by utilizing the following WSDL: <br>
https://za.wigroup.co:8181/wigroup-transactionengine/PosProviderWS?wsdl
</aside>

An integrated mobile application will generate a transaction token (wiCode) which is scanned by the point of sale and then sent to the wiCode platform via a web-service over an SSL connection. The platform will route the transaction to whichever mobile application issued the wiCode for authorisation, and will then send the response back to the POS.


<aside class="success">
The destination IP is 54.76.23.107, using either a TCP port (8080) or an SLL port connection (8181) for production. The associated DNS is za.wigroup.co.
</aside>

# Transaction

Examples of each of the API calls are presented with a description of the objects and the fields.

## Layout

### Transaction Request

> The basic layout of the transaction <b>request</b> looks as follows:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" 
xmlns:pos="http://posprovider.te.wigroup.com/">
  <soapenv:Header/>
  <soapenv:Body>
    <pos:transaction>
      <request>
        <apiCredentials>
          <id></id>
          <password></password>
        </apiCredentials>
        <basketAmount></basketAmount>
        <cashbackAmount></cashbackAmount>
        <products>
          <product>
            <id></id>
            <pricePerUnit></pricePerUnit>
            <units></units>
          </product>
          <product>
            <id></id>
            <pricePerUnit></pricePerUnit>
            <units></units>
          </product>
        </products>
        <storeTrxDetails>
          <basketId></basketId>
          <cashierId></cashierId>
          <posId></posId>
          <remoteStoreId></remoteStoreId>
          <retailerId></retailerId>
          <storeId></storeId>
          <trxId></trxId>
        </storeTrxDetails>
        <switchTrxId></switchTrxId>
        <tipAmount></tipAmount>
        <token>
          <id></id>
          <type></type>
        </token>
        <totalAmount></totalAmount>
        <type></type>
      </request>
    </pos:transaction>
  </soapenv:Body>
</soapenv:Envelope>
```

```java
public class posIntegration{

  // wiGroup Transaction Engine location
  private static final String te_URI = 
   "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl";
       
  private static Long doTransaction(String wiCode, StoreTrxDetails storeTrxDetails, PosProviderWS_Service stub) throws MalformedURLException{
    ApiCredentials apiCredentials = new ApiCredentials();
    apiCredentials.setId(posApiId);
    apiCredentials.setPassword(posApiPassword);
        
    Token token = new Token();
    token.setId(wiCode);
    token.setType(posTokenType);
    
    Product prod1 = new Product();
    prod1.setId(prod1ID);
    prod1.setPricePerUnit(prod1Price);
    prod1.setUnits(prod1Price);
    
    Product prod2 = new Product();
    prod2.setId(prod2ID);
    prod2.setPricePerUnit(prod2Price);
    prod2.setUnits(prod2Price);

    PosProviderTransactionRequest transactionRequest = 
      new PosProviderTransactionRequest();
    transactionRequest.setApiCredentials(apiCredentials);
    transactionRequest.setStoreTrxDetails(storeTrxDetails);
    transactionRequest.setToken(token);
    transactionRequest.setBasketAmount(basketAmount);
    transactionRequest.setCashbackAmount(cashbackAmount);
    transactionRequest.setTipAmount(tipAmount);
    transactionRequest.setTotalAmount(totalAmount);
    transactionRequest.setType(posType);      
    transactionRequest.getProduct().add(prod1);
    transactionRequest.getProduct().add(prod2);
        
    PosProviderTransactionResponse response =
      stub.getPosProviderWSPort().transaction(transactionRequest);
        
    System.out.println("\n\nSome important information on the Transaction\nwiCode: " + wiCode + 
      "\nResponse code: " + response.getResponseCode() +
      "\nResponse description: " + response.getResponseDesc() +
      "\nwiTrxId: "+ response.getWiTrxId() +
      "\nDiscount: "   );
        
    for(int i = 0; i < response.getDiscount().size(); i++) {
      System.out.println("\tamount: " + response.getDiscount().get(i).getAmount());
    }
        
    wiTrxId = response.getWiTrxId();
    trxId= response.getVsp().getTrxId();
        
    if("-1".equals(response.getResponseCode()))
      return (long) -1;
    else
     return response.getWiTrxId();
  }   
    
  public static void main(String[] args) throws MalformedURLException, IOException, JSONException {
    // Creating the store details for the transaction
    StoreTrxDetails storeTrxDetails = new StoreTrxDetails();
    storeTrxDetails.setBasketId(posBasketId);
    storeTrxDetails.setCashierId(posCashierID);
    storeTrxDetails.setPosId(posPosId);
    storeTrxDetails.setStoreId(posStoreId);
    storeTrxDetails.setTrxId(posTrxId);
        
    PosProviderWS_Service stub = new PosProviderWS_Service(new URL(te_URI));
        
    // Performing the transaction
    doTransaction(wiCode, storeTrxDetails, stub);
  }
}
```

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title></title>
  </head>
    <?php
    class apiCredentials {
      function apiCredentials($id, $password) 
      {
        $this->apiId = $id;
        $this->apiPassword = $password;
      }
    }
    class storeTrxDetails {
      function storeTrxDetails($basket, $cashier, $pos, $store, $trx) 
      {
        $this->basketId = $basket;
        $this->cashierId = $cashier;
        $this->posId = $pos;
        $this->storeId = $store;
        $this->trxId = $trx;
      }
    }
    class token {
      function token($id, $type) 
      {
        $this->id = $id;
        $this->type = $type;
      }
    }
    class product {
      function product($id, $pricePerUnit, $units) 
      {
        $this->id = $id;
        $this->pricePerUnit = $pricePerUnit;
        $this->units = $units;
      }
    }

    $url = "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl";

    $transactionRequest = "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:pos=\"http://posprovider.te.wigroup.com/\">
      <soapenv:Header/>
      <soapenv:Body>
       <pos:transaction>
         <request>
          <apiCredentials>
            <id>".$apiCredentials->apiId."</id>
            <password>".$apiCredentials->apiPassword."</password>
          </apiCredentials>
          <product>
            <id>".$prod1->id."</id>
            <pricePerUnit>".$prod1->pricePerUnit."</pricePerUnit>
            <units>".$prod1->units."</units>
          </product>
          <product>
            <id>".$prod2->id."</id>
            <pricePerUnit>".$prod2->pricePerUnit."</pricePerUnit>
            <units>".$prod2->units."</units>
          </product>
          <storeTrxDetails>
            <basketId>".$storeTrxDetails->basketId."</basketId>
            <cashierId>".$storeTrxDetails->cashierId."</cashierId>
            <posId>".$storeTrxDetails->posId."</posId>
            <storeId>".$storeTrxDetails->storeId."</storeId>
            <trxId>".$storeTrxDetails->trxId."</trxId>
          </storeTrxDetails>
          <basketAmount>".$basketAmount."</basketAmount>
          <token>
            <id>".$token->id."</id>
            <type>".$token->type."</type>
          </token>
          <totalAmount>".$totalAmount."</totalAmount>
          <type>".$type."</type>
         </request>
       </pos:transaction>
      </soapenv:Body>";

    $header = array(
      "Content-type: text/xml;charset=\"utf-8\";application/xml",
      "Accept: text/xml",
      "Cache-Control: no-cache",
      "Pragma: no-cache",
      "SOAPAction: \"run\"",
      "Content-length: ".strlen($transactionRequest),
    );

    $soap_do = curl_init();
    curl_setopt($soap_do, CURLOPT_URL, "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl");
    curl_setopt($soap_do, CURLOPT_CONNECTTIMEOUT, 10);
    curl_setopt($soap_do, CURLOPT_TIMEOUT, 30);
    curl_setopt($soap_do, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($soap_do, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($soap_do, CURLOPT_SSL_VERIFYHOST, false);
    curl_setopt($soap_do, CURLOPT_POST, true);
    curl_setopt($soap_do, CURLOPT_POST, true);
    curl_setopt($soap_do, CURLOPT_POSTFIELDS, $transactionRequest);
    curl_setopt($soap_do, CURLOPT_HTTPHEADER, $header);

    $result = curl_exec($soap_do);

    $xml = simplexml_load_string($result);
    $xml->registerXPathNamespace('S', 'http://schemas.xmlsoap.org/soap/envelope/');
    $xml->registerXPathNamespace('ns2', 'http://posprovider.te.wigroup.com/');
    $xml_resp = $xml->xpath('//ns2:transactionResponse');
    $response = $xml_resp[0]->response;
    echo "<h3>Some important information on the transaction:</h3>";
    echo "<p>wiCode: ".$response->token->id."<br>";
    echo "Response code: ".$response->responseCode."<br>";
    echo "Response description: ".$response->responseDesc."<br>";
    echo "wiTrxId: ".$response->wiTrxId."<br>";
    echo "Discount: <br>";
    $discounts = $response->discount;
    foreach($discounts->amount as $amount){
      echo "amount:".$amount."<br>";
    }
  ?>
</html>
```

This call is performed when a transaction is logged in the transaction engine and routed to the relevant VSP. This call usually contains more information than the Advice calls. The basic layout of the transaction request looks a follows:

Fields | Optional or Required | Description
------ | -------------------- | -----------
API Credentials | Required | API credentials issued to you by wiGroup.
BasketAmount | Required | This is the total transaction amount (in cents) of all products in the basket.
CashBackAmount | Optional | This is the amount of cash in cent being withdrawn as part of the transaction. It can either be used in conjunction with a basked payment or on its own. This is only for use with <b><font face="Courier New"> PAYMENT</font></b> type.
Product (Products) <br> <em>ID</em> <br> <em>PricePerUnit</em> <br> <em>Units</em> | Required | The ID refers to the actual SKU number of the product. The price per unit at the POS and the number of units. It is recommended that the POS provides a list of products for each transaction.
StoreTrxDetails <br> <em>BasketId</em> <br> <em>CashierId</em> <br> <em>PosID</em> <br> <em>RemoteStoreID</em> <br> <em>RetailerID</em> <br> <em>StoreID</em> <br> <em>TrxID</em> | <br> Required <br> Required <br> Required <br> Optional <br> Optional <br> Optional <br> Required |  <br> These are the internal retailer ID's for the basket, cashier and POS. <br> Store details: <br> StoreID: is the store WID assigned to you by wiGroup. If the StoreID is populated, this will be used. Alternatively, the RemoteStoreID and the RetailerID must be populated - these are used in combination. <br> TrxID:  The internal retailers unique transaction ID
SwitchTrxID | Optional | If applicable, the switch will assign their ID to the transaction.
TipAmount | Optional | The tip amount must be included in the total amount in order for the VSP to process the full amount. Not all VSP's support tip amount.
Token <br> <em>ID</em> <br> <em>Type</em> | Required | The ID is the unique token which the wiPlatform uses to identify and route the transaction to the VSP to authorize. The token type is derived from how the token is received at the POS and can be either <b><font face="Courier New"> WICODE</font></b>, <b><font face="Courier New"> WIQR </font></b> or <b><font face="Courier New"> BIN</font></b>.
TotalAmount | Required | This is the total transaction amount in cent. This is the sum of the BasketAmount, the CashbackAmount, and TipAmount.
Type | Required | This is the type of transaction. It can set to either <b><font face="Courier New">PAYMENT</font></b>, <b><font face="Courier New"> DEPOSIT </font></b> or <b><font face="Courier New"> WITHDRAWAL</font></b>.

<aside class="notice">
There are three transaction types that are currently supported by the wiCode Platform. <ul> <li><b><font face="Courier New">PAYMENT</font></b> <li><b><font face="Courier New">WITHDRAWAL</font></b>  <li><b><font face="Courier New">DEPOSIT </font></b></ul> 
</aside>

### Transaction Response

> The basic layout of the transaction <b>response</b> looks as follows:

```xml
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:transactionResponse xmlns:ns2="http://posprovider.te.wigroup.com/">
      <response>
        <responseCode></responseCode>	
        <responseDesc></responseDesc>
        <basketAmountProcessed></basketAmountProcessed>
        <cashbackAmountProcessed></cashbackAmountProcessed>
        <storeTrxDetails>
          <basketId></basketId>
          <cashierId></cashierId>
          <posId></posId>
          <remoteStoreId></remoteStoreId>
          <storeId></storeId>
          <trxId></trxId>
        </storeTrxDetails>
        <tipAmountProcessed></tipAmountProcessed>
        <token>
          <id></id>
          <type></type>
        </token>
        <totalAmountProcessed></totalAmountProcessed>
        <type></type>
        <vsp>
          <id></id>
          <message></message>
          <name></name>
          <responseCode></responseCode>
          <responseDesc></responseDesc>
          <trxId></trxId>
        </vsp>
        <wiTrxId></wiTrxId>
        <discount>
          <com.wigroup.te.common.pojo.Discount>
            <amount></amount>
            <product/>
          </com.wigroup.te.common.pojo.Discount>
          <com.wigroup.te.common.pojo.Discount>
            <name></name>
            <amount></amount>
            <product>
              <com.wigroup.te.common.pojo.DiscountProduct>
                <id></id>
                <units></units>
              </com.wigroup.te.common.pojo.DiscountProduct>
            </product>
          </com.wigroup.te.common.pojo.Discount>
        </discount>
      </response>
    </ns2:transactionResponse>
  </S:Body>
</S:Envelope>
```

```java
// Some important information on the transaction
wiCode: 1122345
Response code: -1
Response description: Success
wiTrxId: 11222
Discount: 
  amount: 500
  amount: 1000
```

```php
// Some important information on the transaction
wiCode: 1122345
Response code: -1
Response description: Success
wiTrxId: 11222
Discount: 
  amount: 500
  amount: 1000
```

Each transaction type uses the XML in a (slightly) different manner.

Fields | Description
------ | -----------
ResponseCode & ResponseDescription | Code and description of the code. Use the VSP response for further details to display to user. A code of “-1” will always be associated with a “Success” description.
BasketAmountProcessed | The basket amount that was used in processing the transaction.
CashBackAmount | This is the amount of cash in cents being withdrawn as part of the transaction. It can either be used in conjunction with a basket payment or on its own.
StoreTrxDetails | Returning the same details that were sent up in the request
Token <br> <em>ID, Type</em> | Returning the same details that were sent up in the request. 
TotalAmountProcessed | This is the total transaction amount in cent. This is the sum of the BasketAmount, the CashbackAmount, and TipAmount.
Type | The transaction type that was processed. It can set to either <b><font face="Courier New">PAYMENT</font></b>, <b><font face="Courier New"> DEPOSIT </font></b> or <b><font face="Courier New"> WITHDRAWAL</font></b>.
VSP <br><em>ID </em> <br> <em>Message </em> <br <em>ResponseCode </em> <br> <em>ResponseDescription </em> <br> <em>TrxID </em> | Details indicate what VSP was used to process the transaction and the VSP response code. <br> The response description can be used to display to the user / customer. <br> The unique VSP transaction ID for the requested transaction is presented by TrxID. 
wiTrxID | This is the unique wiGroup transaction ID assigned to the transaction. This field is important for processing a subsequent <b><font face="Courier New">ADVICE </font></b> call.
Discount <br> <em>Amount</em> <br> <u><em>Product</em></u> <br>   <em>Id</em> <br>    <em>Units</em> | All the items that was discounted on the transaction. <br> This returned by a discount VSP and is usually from the wiGroup Coupon Voucher Server. When product information is provided, a coupon has been redeemed, and when no product information is present, a voucher has been redeemed.

## Payment



> The basic layout of the <b><font face="Courier New">PAYMENT</font></b> <b>request</b> looks as follows:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" 
xmlns:pos="http://posprovider.te.wigroup.com/">
  <soapenv:Header/>
  <soapenv:Body>
    <pos:transaction>
      <request>
        <apiCredentials>
          <id>paymentApiId</id>
          <password>paymentApiPassword</password>
        </apiCredentials>
        <basketAmount>1500</basketAmount>
        <product>
          <id>1111</id>
          <pricePerUnit>1000</pricePerUnit>
          <units>1</units>
        </product>
        <product>
          <id>2222</id>
          <pricePerUnit>250</pricePerUnit>
          <units>2</units>
        </product>
        <storeTrxDetails>
          <basketId>basket002</basketId>
          <cashierId>cashier002</cashierId>
          <posId>pos002</posId>
          <remoteStoreId>remoteStore002</remoteStoreId>
          <retailerId>retailer002</retailerId>
          <storeId>1050</storeId>
          <trxId>trx002</trxId>
        </storeTrxDetails>
        <switchTrxId>switchTrx002</switchTrxId>
        <token>
          <id>1234567</id>
          <type>WICODE</type>
        </token>
        <totalAmount>1500</totalAmount>
        <type>PAYMENT</type>
      </request>
    </pos:transaction>
  </soapenv:Body>
</soapenv:Envelope>
```

```java
public class posIntegration{

  // wiGroup Transaction Engine location
  private static final String te_URI = 
   "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl";
       
  private static Long doTransaction(String wiCode, StoreTrxDetails storeTrxDetails, PosProviderWS_Service stub) throws MalformedURLException{
    ApiCredentials apiCredentials = new ApiCredentials();
    apiCredentials.setId("paymentApiId");
    apiCredentials.setPassword("paymentApiPassword");
        
    Token token = new Token();
    token.setId("1234567");
    token.setType("WICODE");

    PosProviderTransactionRequest transactionRequest = 
      new PosProviderTransactionRequest();
    transactionRequest.setApiCredentials(apiCredentials);
    transactionRequest.setStoreTrxDetails(storeTrxDetails);
    transactionRequest.setToken(token);
    
    Product prod1 = new Product();
    prod1.setId("1111");
    prod1.setPricePerUnit(1000);
    prod1.setUnits(1);
    
    Product prod2 = new Product();
    prod2.setId("2222");
    prod2.setPricePerUnit(250);
    prod2.setUnits(2);
    
    transactionRequest.getProduct().add(prod1);
    transactionRequest.getProduct().add(prod2);
    
      // This is a R15.00 transaction
    transactionRequest.setBasketAmount(1500);
    transactionRequest.setTotalAmount(1500);
    transactionRequest.setType("PAYMENT");      
        
    PosProviderTransactionResponse response =
      stub.getPosProviderWSPort().transaction(transactionRequest);
        
    System.out.println("\n\nSome important information on the transaction\nwiCode: " + wiCode + 
      "\nResponse code: " + response.getResponseCode() +
      "\nResponse description: " + response.getResponseDesc() +
      "\nwiTrxId: "+ response.getWiTrxId() +
      "\nDiscount: "   );
        
    for(int i = 0; i < response.getDiscount().size(); i++) {
      System.out.println("\tamount: " + response.getDiscount().get(i).getAmount());
    }
        
    wiTrxId = response.getWiTrxId();
    trxId= response.getVsp().getTrxId();
        
    if("-1".equals(response.getResponseCode()))
      return (long) -1;
    else
     return response.getWiTrxId();
  }   
    
  public static void main(String[] args) throws MalformedURLException, IOException, JSONException {
    // Creating the store details for the transaction
    StoreTrxDetails storeTrxDetails = new StoreTrxDetails();
    storeTrxDetails.setBasketId("basket002");
    storeTrxDetails.setCashierId("cashier002");
    storeTrxDetails.setPosId("pos002");
    storeTrxDetails.setStoreId(1050);
    storeTrxDetails.setTrxId("trx002");
        
    PosProviderWS_Service stub = new PosProviderWS_Service(new URL(te_URI));
        
    // Performing the transaction
    doTransaction(wiCode, storeTrxDetails, stub);
  }
}
```

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title></title>
  </head>
  <?php
    class apiCredentials {
      function apiCredentials($id, $password) 
      {
        $this->apiId = $id;
        $this->apiPassword = $password;
      }
    }
    class storeTrxDetails {
      function storeTrxDetails($basket, $cashier, $pos, $store, $trx) 
      {
        $this->basketId = $basket;
        $this->cashierId = $cashier;
        $this->posId = $pos;
        $this->storeId = $store;
        $this->trxId = $trx;
      }
    }
    class token {
      function token($id, $type) 
      {
        $this->id = $id;
        $this->type = $type;
      }
    }
    class product {
      function product($id, $pricePerUnit, $units) 
      {
        $this->id = $id;
        $this->pricePerUnit = $pricePerUnit;
        $this->units = $units;
      }
    }

    $apiCredentials = new apiCredentials("test", "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3");
    $storeTrxDetails = new storeTrxDetails("aa", "112aa2", "112aa2", "1050", "11aa22");
    $token = new token("9366873", "WICODE");
    $basketAmount = 3000;
    $totalAmount = 3000;
    $type = "PAYMENT";
    $prod1 = new product(1111, 1000, 1);
    $prod2 = new product(2222, 250, 2);
    $url = "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl";

    $transactionRequest = "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:pos=\"http://posprovider.te.wigroup.com/\">
      <soapenv:Header/>
      <soapenv:Body>
       <pos:transaction>
         <request>
          <apiCredentials>
            <id>".$apiCredentials->apiId."</id>
            <password>".$apiCredentials->apiPassword."</password>
          </apiCredentials>
          <product>
            <id>".$prod1->id."</id>
            <pricePerUnit>".$prod1->pricePerUnit."</pricePerUnit>
            <units>".$prod1->units."</units>
          </product>
          <product>
            <id>".$prod2->id."</id>
            <pricePerUnit>".$prod2->pricePerUnit."</pricePerUnit>
            <units>".$prod2->units."</units>
          </product>
          <storeTrxDetails>
            <basketId>".$storeTrxDetails->basketId."</basketId>
            <cashierId>".$storeTrxDetails->cashierId."</cashierId>
            <posId>".$storeTrxDetails->posId."</posId>
            <storeId>".$storeTrxDetails->storeId."</storeId>
            <trxId>".$storeTrxDetails->trxId."</trxId>
          </storeTrxDetails>
          <basketAmount>".$basketAmount."</basketAmount>
          <token>
            <id>".$token->id."</id>
            <type>".$token->type."</type>
          </token>
          <totalAmount>".$totalAmount."</totalAmount>
          <type>".$type."</type>
         </request>
       </pos:transaction>
      </soapenv:Body>";

    $transactionHeader = array(
      "Content-type: text/xml;charset=\"utf-8\";application/xml",
      "Accept: text/xml",
      "Cache-Control: no-cache",
      "Pragma: no-cache",
      "SOAPAction: \"run\"",
      "Content-length: ".strlen($transactionRequest),
    );

    $performTransaction = curl_init();
    curl_setopt($performTransaction, CURLOPT_URL, $url);
    curl_setopt($performTransaction, CURLOPT_CONNECTTIMEOUT, 10);
    curl_setopt($performTransaction, CURLOPT_TIMEOUT, 30);
    curl_setopt($performTransaction, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($performTransaction, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($performTransaction, CURLOPT_SSL_VERIFYHOST, false);
    curl_setopt($performTransaction, CURLOPT_POST, true);
    curl_setopt($performTransaction, CURLOPT_POST, true);
    curl_setopt($performTransaction, CURLOPT_POSTFIELDS, $transactionRequest);
    curl_setopt($performTransaction, CURLOPT_HTTPHEADER, $transactionHeader);

    $transactionResult = curl_exec($performTransaction);

    // echo $result;

    $transactionXml = simplexml_load_string($transactionResult);
    $transactionXml->registerXPathNamespace('S', 'http://schemas.xmlsoap.org/soap/envelope/');
    $transactionXml->registerXPathNamespace('ns2', 'http://posprovider.te.wigroup.com/');
    $xml_resp = $transactionXml->xpath('//ns2:transactionResponse');
    $transactionResponse = $xml_resp[0]->response;
    echo "<h3>Some important information on the transaction:</h3>";
    echo "<p>wiCode: ".$transactionResponse->token->id."<br>";
    echo "Response code: ".$transactionResponse->responseCode."<br>";
    echo "Response description: ".$transactionResponse->responseDesc."<br>";
    echo "wiTrxId: ".$transactionResponse->wiTrxId."<br>";
    echo "Discount: <br>";
    $discounts = $transactionResponse->discount;
    foreach($discounts->amount as $amount){
      echo "amount:".$amount."<br>";
    }
  ?>
</html>
```

The <b><font face="Courier New">PAYMENT</font></b> type transaction request is used most often. Transaction request of the type <b><font face="Courier New">PAYMENT</font></b> should be used for all payment related transactions as well as the redemption of wiGroup coupon and vouchers.



It is important to note that the field totalAmountProcessed in the transaction response is not necessarily equal to value set in the basketAmount in the transaction request. A simple explanation of this significance is to consider a customer attempting to settle a bill worth R30.00 with a wiCode voucher worth R25.00. The POS may specify the basketAmount to be 3000 in the transaction request. The response will then provide the totalAmountProcessed to be 2500. The POS will then have to ensure that another tender of R5.00 takes place in order to settle the bill. Therefore, the POS must allow for multi-tender and split tender for transactions on the wiCode platform.





> The basic layout of the <b><font face="Courier New">PAYMENT</font></b> <b>response</b> looks as follows:

```xml
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:transactionResponse xmlns:ns2="http://posprovider.te.wigroup.com/">
      <response>
        <responseCode>-1</responseCode>
        <responseDesc>Success</responseDesc>
        <basketAmountProcessed>1500</basketAmountProcessed>
        <storeTrxDetails>
          <basketId>basket002</basketId>
          <cashierId>cashier002</cashierId>
          <posId>pos002</posId>
          <remoteStoreId>remoteStore002</remoteStoreId>
          <storeId>1050</storeId>
          <trxId>trx002</trxId>
        </storeTrxDetails>
        <switchTrxId>switchTrx002</switchTrxId>
        <tipAmountProcessed>0</tipAmountProcessed>
        <token>
          <id>1234567</id>
          <type>WICODE</type>
        </token>
        <totalAmountProcessed>1500</totalAmountProcessed>
        <type>PAYMENT</type>
        <vsp>
          <id>12345</id>
          <message>Success</message>
          <name>testVSP</name>
          <responseCode>-1</responseCode>
          <responseDesc>Success</responseDesc>
          <trxId>907</trxId>
        </vsp>
        <discount>
          <com.wigroup.te.common.pojo.Discount>
            <amount>500</amount>
            <product/>
          </com.wigroup.te.common.pojo.Discount>
          <com.wigroup.te.common.pojo.Discount>
            <name>R2 Voucher</name>
            <amount>200</amount>
            <product>
              <com.wigroup.te.common.pojo.DiscountProduct>
                <id>1111</id>
                <units>1</units>
              </com.wigroup.te.common.pojo.DiscountProduct>
            </product>
          </com.wigroup.te.common.pojo.Discount>
        </discount>
        <wiTrxId>19144</wiTrxId>
      </response>
    </ns2:transactionResponse>
  </S:Body>
</S:Envelope>
```

```java
// Some important information on the transaction
wiCode: 1234567
Response code: -1
Response description: Success
wiTrxId: 12345
Discount: 
  amount: 500
  amount: 200
```

```php
// Some important information on the transaction
wiCode: 1234567
Response code: -1
Response description: Success
wiTrxId: 12345
Discount: 
  amount: 500
  amount: 200
```

## Deposit



> The basic layout of the <b><font face="Courier New">DEPOSIT</font></b> <b>request</b> is similar to that of a the <b><font face="Courier New">PAYMENT</font></b> request. The type requires to be changed accordingly.

```xml
  <type>DEPOSIT</type>
```

```java
  transactionRequest.setType("DEPOSIT");
```

```php
  $type = "DEPOSIT";
```

The <b><font face="Courier New">DEPOSIT</font></b> type transaction request is used to upload funds to a mobile wallet.

## Withdrawal

> The basic layout of the <b><font face="Courier New">WITHDRAWAL</font></b> <b>request</b> is similar to that of a the <b><font face="Courier New">PAYMENT</font></b> request. The type requires to be changed accordingly.

```xml
  <type>WITHDRAWAL</type>
```

```java
  transactionRequest.setType("WITHDRAWAL");
```

```php
  $type = "WITHDRAWAL";
```

The <b><font face="Courier New">WITHDRAWAL</font></b> type transaction request is used to withdraw funds from a mobile wallet.

# Advice

### Advice request

> The basic layout of the <b><font face="Courier New">ADVICE</font></b> <b>request</b> looks as follows:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" 
xmlns:pos="http://posprovider.te.wigroup.com/">
  <soapenv:Header/>
  <soapenv:Body>
   <pos:advise>
     <request>
      <apiCredentials>
        <id>finaliseApiID</id>
        <password>finaliseApiPassword</password>
      </apiCredentials>
      <action>FINALISE</action>
      <originalTrx>
        <storeTrxDetails>
         <basketId>basket005</basketId>
         <cashierId>cashier005</cashierId>
         <posId>pos005</posId>
         <storeId>1050</storeId>
         <trxId>trx005</trxId>
        </storeTrxDetails>
       <type>PAYMENT</type>
       <vspTrxId>862</vspTrxId>
       <wiTrxId>17956</wiTrxId>
       </originalTrx>
     </request>
   </pos:advise>
  </soapenv:Body>
</soapenv:Envelope>
```
```java
public class posIntegration{

  // wiGroup Transaction Engine location
  private static final String te_URI = 
   "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl";
       
  private static void doAdvise(Long wiTrxId, StoreTrxDetails storeTrxDetails) throws MalformedURLException{
    PosProviderTransactionAdviceRequest request = new PosProviderTransactionAdviceRequest();
    
    ApiCredentials apiCredentials = new ApiCredentials();
    apiCredentials.setId("finaliseApiID");
    apiCredentials.setPassword("finaliseApiPassword");
    
    request.setApiCredentials(apiCredentials);
    
    OriginalTrx adviceOriginalTrx = new OriginalTrx();
    adviceOriginalTrx.setType("PAYMENT");
    adviceOriginalTrx.setVspTrxId(862);
    adviceOriginalTrx.setWiTrxId(wiTrxId);
    adviceOriginalTrx.setStoreTrxDetails(storeTrxDetails);
    
    PosProviderTransactionAdviceRequest adviceRequest = new PosProviderTransactionAdviceRequest();
    adviceRequest.setApiCredentials(apiCredentials);
    adviceRequest.setAction("FINALISE");
    adviceRequest.setOriginalTrx(adviceOriginalTrx);

    PosProviderWS_Service stub = new PosProviderWS_Service(new URL("http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl"));
    
    PosProviderTransactionAdviceResponse response = stub.getPosProviderWSPort().advise(adviceRequest);
    
    System.out.println("\n\nSome important information on the advise" + 
      "\nResponse code: " + response.getResponseCode() +
      "\nResponse description: " + response.getResponseDesc() +
      "\naction: "+ response.getAction());
  }
    
  public static void main(String[] args) throws MalformedURLException, IOException, JSONException {
    // Store details for the transaction
    StoreTrxDetails storeTrxDetails = new StoreTrxDetails();
    storeTrxDetails.setBasketId("basket005");
    storeTrxDetails.setCashierId("cashier005");
    storeTrxDetails.setPosId("pos005");
    storeTrxDetails.setStoreId(1050);
    storeTrxDetails.setTrxId("trx005");
        
    PosProviderWS_Service stub = new PosProviderWS_Service(new URL(te_URI));
        
    // Performing the advice
    doAdvise(17956, storeTrxDetails);
  }
}
```

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title></title>
  </head>
  
  <?php
    class apiCredentials {
      function apiCredentials($id, $password) 
      {
        $this->apiId = $id;
        $this->apiPassword = $password;
      }
    }

    class storeTrxDetails {
      function storeTrxDetails($basket, $cashier, $pos, $store, $trx) 
      {
        $this->basketId = $basket;
        $this->cashierId = $cashier;
        $this->posId = $pos;
        $this->storeId = $store;
        $this->trxId = $trx;
      }
    }
    class token {
      function token($id, $type) 
      {
        $this->id = $id;
        $this->type = $type;
      }
    }
    class product {
      function product($id, $pricePerUnit, $units) 
      {
        $this->id = $id;
        $this->pricePerUnit = $pricePerUnit;
        $this->units = $units;
      }
    }

    $apiCredentials = new apiCredentials("test", "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3");
    $storeTrxDetails = new storeTrxDetails("aa", "112aa2", "112aa2", "1050", "11aa22");
    $token = new token("706313626", "WICODE");
    $basketAmount = 3000;
    $totalAmount = 3000;
    $type = "PAYMENT";
    $prod1 = new product(1111, 1000, 1);
    $prod2 = new product(2222, 250, 2);
    $url = "http://rad2.wigroup.co:8080/wigroup-te-za-rad/PosProviderWS?wsdl"; 
    $action = "FINALISE";
    
    $adviseRequest = "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:pos=\"http://posprovider.te.wigroup.com/\">
    <soapenv:Header/>
      <soapenv:Body>
        <pos:advise>
          <request>
            <apiCredentials>
              <id>".$apiCredentials->apiId."</id>
              <password>".$apiCredentials->apiPassword."</password>
            </apiCredentials>
            <action>".$action."</action>
            <originalTrx>
              <storeTrxDetails>
                <basketId>".$storeTrxDetails->basketId."</basketId>
                <cashierId>".$storeTrxDetails->cashierId."</cashierId>
                <posId>".$storeTrxDetails->posId."</posId>
                <storeId>".$storeTrxDetails->storeId."</storeId>
                <trxId>".$storeTrxDetails->trxId."</trxId>  
              </storeTrxDetails>
              <type>PAYMENT</type>
              <vspTrxId>".$transactionResponse->vsp->id."</vspTrxId>
              <wiTrxId>".$transactionResponse->wiTrxId."</wiTrxId>
            </originalTrx>
          </request>
        </pos:advise>
      </soapenv:Body>
    </soapenv:Envelope>";

    $adviseHeader = array(
      "Content-type: text/xml;charset=\"utf-8\";application/xml",
      "Accept: text/xml",
      "Cache-Control: no-cache",
      "Pragma: no-cache",
      "SOAPAction: \"run\"",
      "Content-length: ".strlen($adviseRequest),
    );

    $performAdvise = curl_init();
    curl_setopt($performAdvise, CURLOPT_URL, $url);
    curl_setopt($performAdvise, CURLOPT_CONNECTTIMEOUT, 10);
    curl_setopt($performAdvise, CURLOPT_TIMEOUT, 30);
    curl_setopt($performAdvise, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($performAdvise, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($performAdvise, CURLOPT_SSL_VERIFYHOST, false);
    curl_setopt($performAdvise, CURLOPT_POST, true);
    curl_setopt($performAdvise, CURLOPT_POST, true);
    curl_setopt($performAdvise, CURLOPT_POSTFIELDS, $adviseRequest);
    curl_setopt($performAdvise, CURLOPT_HTTPHEADER, $adviseHeader);

    $adviseResult = curl_exec($performAdvise);

    $adviseXml = simplexml_load_string($adviseResult);
    $adviseXml->registerXPathNamespace('S', 'http://schemas.xmlsoap.org/soap/envelope/');
    $adviseXml->registerXPathNamespace('ns2', 'http://posprovider.te.wigroup.com/');
    $xml_resp1 = $adviseXml->xpath('//ns2:adviseResponse');
    $adviseResponse = $xml_resp1[0]->response;
    echo "<h3>Some important information on the advise:</h3>";
    echo "<p>wiCode: ".$transactionResponse->token->id."<br>";
    echo "Response code: ".$adviseResponse->responseCode."<br>";
    echo "Response description: ".$adviseResponse->responseDesc."<br>";
    echo "Action: ".$adviseResponse->action."</p>"; 
  ?>
</html>
```





The advice request must be sent after the transaction has completed at POS (tender is completed). The advice request determines whether the state of a pending transaction is set to either to complete the transaction or to cancel the entire bill. This is done by means of setting the action field to <b><font face="Courier New">REVERSE</font></b> or <b><font face="Courier New">FINALISE</font></b>. This will change the transaction from pending to finalised / reversed state on the wiGroup platform. The POS will receive a response from wiGroup as the final leg of the transaction. It is important to note that the <font face="Courier New">wiTrxId</font> received in the <b>transaction response</b> should be carried over and used the <b>advice request</b> in order to advise that transaction.



This call is performed when a transaction is logged in the transaction engine and routed to the relevant VSP. This call usually contains more information than the Advice calls. The basic layout of the transaction request looks a follows:

Fields | Optional or Required | Description
------ | -------------------- | -----------
API Credentials | Required | API credentials issued to you by wiGroup.
Action | Required | Must be either <b><font face="Courier New">FINALISE</font></b> or <b><font face="Courier New">REVERSE</font></b>.
StoreTrxDetails <br> <em>BasketId</em> <br> <em>CashierId</em> <br> <em>PosID</em> <br> <em>RemoteStoreID</em> <br> <em>RetailerID</em> <br> <em>StoreID</em> <br> <em>TrxID</em> | <br> Required <br> Required <br> Required <br> Optional <br> Optional <br> Optional <br> Required | <br> These are the internal retailer ID's for the basket, cashier and POS. <br> Store details: <br> StoreID: is the store WID assigned to you by wiGroup. If the StoreID is populated, this will be used. Alternatively, the RemoteStoreID and the RetailerID must be populated - these are used in combination. <br> TrxID:  The internal retailers unique transaction ID
SwitchTrxID | Optional | If applicable, the switch will assign their ID to the transaction.
VspTrxId | Optional | The transaction ID supplied by the VSP.
WiTrxID | Required | The transaction ID supplied by the wiCode Platform. <br> This field is returned in the transaction response. This links the transaction and the advice.
Type | Required | This is the type of transaction. It can set to either <b><font face="Courier New">PAYMENT</font></b>, <b><font face="Courier New"> DEPOSIT </font></b> or <b><font face="Courier New"> WITHDRAWAL</font></b>.

### Advice response

> The basic layout of the <b><font face="Courier New">ADVICE</font></b> <b>response</b> looks as follows:

```xml
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
   <ns2:adviseResponse xmlns:ns2="http://posprovider.te.wigroup.com/">
     <response>
      <responseCode>-1</responseCode>
      <responseDesc>Success</responseDesc>
      <action>FINALISE</action>
      <originalTrx>
        <storeTrxDetails>
         <basketId>basket005</basketId>
         <cashierId>cashier005</cashierId>
         <posId>pos005</posId>
         <storeId>1050</storeId>
         <trxId>trx005</trxId>
        </storeTrxDetails>
        <type>PAYMENT</type>
        <vspTrxId>862</vspTrxId>
        <wiTrxId>17956</wiTrxId>
      </originalTrx>
     </response>
   </ns2:adviseResponse>
  </S:Body>
</S:Envelope>
```

```java
// Some important information on the advise
Response code: -1
Response description: Success
action: FINALISE
```

```php
// Some important information on the advise
Response code: -1
Response description: Success
action: FINALISE
```
Fields | Description
------ | -----------
ResponseCode & ResponseDescription | Code and description of the code. Use the VSP response for further details to display to user. A code of “-1” will always be associated with a “Success” description.
Action | Will be either <b><font face="Courier New">FINALISE</font></b> or <b><font face="Courier New">REVERSE</font></b>.
CashBackAmount | This is the amount of cash in cents being withdrawn as part of the transaction. It can either be used in conjunction with a basket payment or on its own.
StoreTrxDetails | Returning the same details that were sent up in the request
Type | The transaction type that was processed. It can set to either <b><font face="Courier New">PAYMENT</font></b>, <b><font face="Courier New"> DEPOSIT </font></b> or <b><font face="Courier New"> WITHDRAWAL</font></b>.
vspTrxId | The unique VSP transaction ID.  
wiTrxID | This is the unique wiGroup transaction ID assigned to the transaction.

# Fields that the POS should retrieve from a config file

wiGroup suggests that certain information be stored on each POS that may be used to simplify the integration process and keep the data received consistent per POS. The following fields can be specified in a configuration file that may be accessed by the POS:

<ul>
<li>WSDL URL to the wiCode platform</li>
<li>API credentials -- ID and Password</li>
<li>Store details -- StoreID, CashierID, PosID</li>
</ul>

# Technical implementation and minimum requirements
<ul>
<li>The testing environment makes use of a TCP (http) connection, using port 8080. In production, a SSL (https) connection must be used using port 8181. Therefore, your system will need to cater for a secure connection.</li>
<li>It is highly recommended that integration and QA teams preview the API WSDL in a free program called SoapUI. This software may be downloaded from <a href='http://www.soapui.org/'>http://www.soapui.org/</a>. </li>
<li>Attempt and remove all fields that are not being used. </li>
<li>API credentials will be provided to each POS provider by the relevant account manager or integrations specialist. </li>
<li>The POS must allow for multi-tender and split tender on wiCode transactions. </li>
<li>The POS should have built-in logic to ensure that if the <b><font face="Courier New">basketAmount</font></b> of a transaction is less than the <b><font face="Courier New">totalAmountProcessed</font></b>, another tender is required to settle the bill. This is used to ensure that split tender may be processed correctly. </li>
<li>The wiCode platform can accommodate wiCodes have a specific value associated with it (e.g. coupons & vouchers), or wiCodes which may be used for any amount processed. There are also the case where wiCodes may be merged in the sense that it consists of a voucher or coupon, together with a payment component. In this case the amount outstanding after the value of the coupons and vouchers have been deducted from the basket amount will be processed as a payment. </li>
<li>Monetary amounts are both accepted and returned in cents, <em>i.e.</em> R123.45 should be recorded as 12345. For each of the three basic requests a token object is required as input. The token object consists of an ID and a type. The ID is a unique numerical entry which the wiCode platform uses to identify and route the transaction to the VSP to authorize.  </li>
<li>The token type can either be <b><font face="Courier New">WICODE</font></b>, <b><font face="Courier New">WIQR</font></b> or <b><font face="Courier New">BIN</font></b>. Generally, 7-, 9- and 12-digit ID's can be either <b><font face="Courier New">WICODE</font></b> or <b><font face="Courier New">WIQR</font></b>. The type <b><font face="Courier New">BIN</font></b> is generally reserved for IDs exceeding 12-digit lengths. </li>
<li>The wiTrxId in the response of the transaction request must be provided in the subsequent ADVICE request to successfully finalise the transaction. </li>
<li>If basket changes after a wiCodes has been processed (remove an item from the basket on the POS), all processed wiCodes need to be reversed. The wiCode platform supports a dual message architecture and as such a reversal message can be sent up in the event of a cancelled or voided transaction on the POS. It is important to note that the advice transaction request (which will either finalise or reverse the transaction) should not be processed until payment has taken place. </li>
<li>If a customer has multiple wiCodes that is to be redeemed, wiCodes associated with coupons or vouchers should preferably be used first. Thereafter wiCodes that hold no specific additional rewards may be processed, if necessary.  </li>
<li>The amounts tendered against the basket for the wiGroup redemption in tender mode must include VAT. Any amount tendered against the basket in tender mode will be “settled” to the merchant and as such will include VAT and can be considered revenue for the store.  </li>
<li>It is recommended that the POS developer studies and becomes familiar with the wiCode platform POS API documentation [2]. </li>
<li>Please refer to wiGroup POS Integration -- User Interface Guide.pdf [3] for guidelines in setting up the User Interface on the POS. </li>
<li>Recon and settlement is catered for as part of the wiCode platform but not a part of this site. Please contact your account manager or integration specialist for further details. </li>
<li>The POS implementation must be identical on all the various POS devices used in the live environment. The POS solutions should be stable over all possible variations of software configuration available.  </li>
</ul>

# Conclusion and notes
This site provides sufficient context of the wiCode platform API document [2], although is not intended as an exhaustive Software Requirement Specification for exact implementation, since nuances between different POS provider software will require slightly different implementations. Please direct any queries with regards to the implementation to your wiGroup account manager or integration specialist. 

# References
<ol>
<li>wiGroup Technical Overview - Overview of the wiCode platform, <a href='./docs/Overview.pdf'>PDF document</a>.</li>
<li>Transaction Engine POS Provider API, version 1.7, <a href='./docs/API.pdf'>PDF document</a>.</li>
<li>wiGroup POS Integration -- User Interface Guide, <a href='./docs/Interface.pdf'>PDF document</a>.</li>
</ol>