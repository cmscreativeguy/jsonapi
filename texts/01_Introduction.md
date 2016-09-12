﻿<a name="intro"></a>Introduction
============

This page describes the Saferpay JSON application programming interface.

Our API is designed to have predictable, resource-oriented URLs and to use HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which can be understood by off-the-shelf HTTP clients. JSON will be returned in all responses from the API, including errors.

--->>>

Base URL production system:

`https://www.saferpay.com/api`

Base URL test system:

`https://test.saferpay.com/api`

Test accounts:

<a target="_blank" href="https://test.saferpay.com/BO/Welcome">Request your personal test account</a><br />
<a target="_blank" href="https://www.six-payment-services.com/en/site/saferpay-support/testaccount.html">View shared test account data</a>

<<<---

<a name="encoding"></a>Content Encoding
================

`UTF-8` must be used for text encoding (there are restrictions on allowed characters for specific fields though).

`Content-Type` and `Accept` headers should be set to `application/json` for server-to-server calls. Redirects use the standard browser types.

--->>>

HTTP Headers:

`Content-Type: application/json; charset=utf-8`

`Accept: application/json`

<<<---

<a name="formats"></a>Formats
=======

The Saferpay JSON Api uses unified and standardized formats. The following abbreviations for format information are used in this page:

<table class="table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Definition</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Id</td>
      <td>
        <samp>A-Za-z0-9.:-_</samp>
      </td>
      <td>Alphanumeric with dot, colon, hyphen and underscore.</td>
    </tr>
    <tr>
      <td>Numeric</td>
      <td>
        <samp>0-9</samp>
      </td>
      <td>Numbers.</td>
    </tr>
    <tr>
      <td>Boolean</td>
      <td>
        <samp>true</samp> or <samp>false</samp>
      </td>
      <td>Boolean values.</td>
    </tr>
    <tr>
      <td>Date</td>
      <td>ISO 8601 Date and Time</td>
      <td>
        ISO 8601 format, e.g. <samp>2007-12-24T18:21:25.123Z</samp> (UTC)
        or <samp>2007-12-24T19:21:25.123+01:00</samp> (CET). Max 3 digits in the fractional seconds part.
      </td>
    </tr>
  </tbody>
</table>

<a name="authentication"></a>Authentication
==============

Saferpay supports the mechanism of basic authentication for authentication of a
server (host) system.

<a name="authentication_basic"></a>HTTP basic authentication
-------------------------

This is the default authentication method. Technical users for the JSON API can be
created by the merchant in the Saferpay Backoffice. The password will not be stored
at SIX (only a securely salted hash thereof). There will be no password recovery
besides creating a new user / password pair from your Backoffice account.

The password must meet some complexity requirements. We suggest using / generating
dedicated passwords with a length of 16 alphanumeric characters (up to 32 characters).

--->>>

HTTP Header:

`Authorization: Basic [your base64 encoded user name + password]`

<<<---

<a name="integration"></a>Integration
===========

The JSON API is a modern and lightweight interface, that can be used with all shop
systems and all programming languages. Only a few steps are neccessary to integrate
your only shop with Saferpay. The proceeding is mostly as follows:

1. Initialize via secure server-to-server call
2. Integrate iframe to redirect your customer
3. Authorize/ assert customer interaction via secure server-to-server call

In secure server-to-server calls you have to submit a JSON request containing
you processing instructions to the defined URLs. The URL and the JSON structure
varies depening on the action/resource you want to call. For further details check
the description of resources below.

Server-to-server calls are a secure way to submit and gather data. Hence, a server-to-server
call should always follow after the customer returns back to the shop, to gather
information about the outcome of e.g. 3D Secure.

--->>>

Server-to-server communication:

<div class="samples" role="tabpanel">
  <ul id="tabs" class="nav nav-tabs" data-tabs="tabs">
    <li class="active">
      <a aria-expanded="false" href="#csharp" data-toggle="tab">C#</a>
    </li>
    <li class="">
      <a aria-expanded="true" href="#java" data-toggle="tab">Java</a>
    </li>
    <li class="">
      <a aria-expanded="true" href="#php" data-toggle="tab">PHP</a>
    </li>
  </ul>
  <div id="tab-content" class="tab-content">
    <div class="tab-pane active" id="csharp">
      <pre class="prettyprint">
private object SubmitRequest(string sfpUrl, object request, string sfpLogin, string sfpPassword)
{
    using (var client = new WebClient())
    {
        string authInfo = string.Format("{0}:{1}", sfpLogin, sfpPassword);
        client.Headers[HttpRequestHeader.Authorization] = "Basic " + Convert.ToBase64String(Encoding.UTF8.GetBytes(authInfo));
        client.Headers[HttpRequestHeader.Accept] = "application/json";
        client.Headers[HttpRequestHeader.ContentType] = "application/json; charset=utf-8";
        client.Encoding = Encoding.UTF8;
        try
        {
            var responseData = client.UploadString(sfpUrl, JsonConvert.SerializeObject(request));
            return JsonConvert.DeserializeObject(responseData);
        }
        catch (WebException)
        {
            Trace.WriteLine("Web exception occured");
            // handle error response here
            throw;
        }
    }
}
      </pre>
    </div>
    <div class="tab-pane" id="java">
      <pre class="prettyprint"> public static JsonObject sendRequest(URL sfpUrl, JsonObject request, String sfpLogin, String sfpPassword) throws IOException {
    //encode credentials
    String credential = sfpLogin + &quot;:&quot; + sfpPassword;
    String encodedCredentials = DatatypeConverter.printBase64Binary(credential.getBytes());

    //create connection
    HttpURLConnection connection = (HttpURLConnection) sfpUrl.openConnection();
    connection.setRequestProperty(&quot;connection&quot;, &quot;close&quot;);
    connection.setRequestProperty(&quot;Content-Type&quot;, &quot;application/json; charset=utf-8&quot;);
    connection.setRequestProperty(&quot;Accept&quot;, &quot;application/json&quot;);
    connection.setRequestProperty(&quot;Authorization&quot;, &quot;Basic &quot; + encodedCredentials);
    connection.setRequestMethod(&quot;POST&quot;);
    connection.setDoOutput(true);
    connection.setUseCaches(false);

    //write JSON to output stream
    JsonWriter writer = Json.createWriter(connection.getOutputStream());
    writer.writeObject(request);
    writer.close();

    //send request
    int responseCode = connection.getResponseCode();

    //get correct input stream
    InputStream readerStream = responseCode == 200 ? connection.getInputStream() : connection.getErrorStream();
    JsonObject response = Json.createReader(readerStream).readObject();

    return response;
  }</pre>
    </div>
    <div class="tab-pane" id="php">
      <pre class="prettyprint">
//JSONObj is a multidimensional Array, that assembles the JSON structure
//$username and $password for the http-Basic Authentication
//$url is the SaferpayURL eg. https://www.saferpay.com/api/Payment/v1/Transaction/Initialize
function do_post($username,$password,$url, $JSONObj){
	//Set Options for CURL
	$curl = curl_init($url);
	curl_setopt($curl, CURLOPT_HEADER, false);
	//Return Response to Application
	curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
	//Set Content-Headers to JSON
	curl_setopt($curl, CURLOPT_HTTPHEADER,array("Content-type: application/json"));
	//Execute call via http-POST
	curl_setopt($curl, CURLOPT_POST, true);
	//Set POST-Body
	//convert DATA-Array into a JSON-Object
	curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($JSONObj));
	//WARNING!!!!!
	//This option should NOT be "false"
	//Otherwise the connection is not secured
	//You can turn it of if you're working on the test-system with no vital data
	curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, true);
	curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 0);
	//HTTP-Basic Authentication for the Saferpay JSON-API
	curl_setopt($curl, CURLOPT_USERPWD, $username . ":" . $password);
	//CURL-Execute &amp; catch response
	$jsonResponse = curl_exec($curl);
	//Get HTTP-Status
	//Abort if Status != 200
	$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
	if ( $status != 200 ) {
        die("Error: call to URL $url failed with status $status, response $json_response, curl_error " . curl_error($curl) . ", curl_errno " . curl_errno($curl) . "HTTP-Status: " . $status . "&lt;||||&gt; DUMP: URL: ". $url . " &lt;|||&gt; JSON: " . var_dump($JSONObj));
	}
	//IMPORTANT!!!
	//Close connection!
	curl_close($curl);
	//Convert response into an Array
	$response = json_decode($jsonResponse, true);
	return $response;
}
      </pre>
    </div>
  </div>
</div>

<<<---

If you include the redirect pages into page using an iframe, you can react on size changes
of the iframe content by listening to a message event containing the new sizing information.

Please note: Depending on the bank, issuer, or payment provider, the page can try to break
out of the iframe or lack telling you the actual size of the content.

--->>>

Handle JavaScript events from Saferpay:

````js
$(window).bind('message', function (e) {
  switch (e.originalEvent.data.message) {
    case 'css':
      $('#iframe').css('height', e.originalEvent.data.height + "px");
      break;
    }
  });
````

<<<---

<a name="errorhandling"></a>Error Handling
==============

Successfully completed requests are confirmed with an http status code of `200` and 
contain the appropriate response message in the body.

If the request could not be completed successfully, this is indicated by a status code 
of `400` or higher and – if possible (some errors are generated by the web server itself, 
or the web application firewall and are thus outside of our control) – an error message 
stating the reason of the failure is included in the body of the response. The presence 
of an error message as specified in this document can be derived from the content type: 
if it’s `application/json`, then there is an error message present.

--->>>

HTTP status codes:

<table class="table table-striped">
	<tr>
		<td class="text-right col-sm-2">200</td>
		<td class="col-sm-10">OK (No error)</td>
	</tr>
	<tr>
		<td class="text-right">400</td>
		<td>Validation error</td>
	</tr>
	<tr>
		<td class="text-right">401</td>
		<td>Authentication of the request failed</td>
	</tr>
	<tr>
		<td class="text-right">402</td>
		<td>Requested action failed</td>
	</tr>
	<tr>
		<td class="text-right">403</td>
		<td>Access denied</td>
	</tr>
	<tr>
		<td class="text-right">406</td>
		<td>Not acceptable (wrong accept header)</td>
	</tr>
	<tr>
		<td class="text-right">415</td>
		<td>Unsupported media type (wrong content-type header)</td>
	</tr>
	<tr>
		<td class="text-right">500</td>
		<td>Internal error</td>
	</tr>
</table>

<<<---

#### Error Message




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Behavior</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">What can be done to resolve the error?</div>
	<i class="small text-muted">
Possible values: ABORT, OTHER_MEANS, RETRY, RETRY_LATER.<br />
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ErrorName</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name / id of the error. These names will not change, so you may parse these and attach your logic to the ErrorName.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ErrorMessage</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Description of the error. The contents of this element might change without notice, so do not parse it.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id of the failed transaction, if available</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ErrorDetail</strong><br />
	<span class="text-muted small">
		string[]
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">More details, if available. Contents may change at any time, so don’t parse it.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ProcessorName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of acquirer (if declined by acquirer) or processor</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ProcessorResult</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Result code returned by acquirer or processor</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ProcessorMessage</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Message returned by acquirer or processor</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Behavior": "ABORT",
  "ErrorName": "VALIDATION_FAILED",
  "ErrorMessage": "Request validation failed",
  "ErrorDetail": [
    "PaymentMeans.BankAccount.IBAN: The field IBAN is invalid."
  ]
}
</pre>

<<<---




--->>>

List of behaviors:

<table class="table table-striped">
	<tr>
		<td class="text-right col-sm-3">ABORT</td>
		<td class="col-sm-9">Do not retry this request. It will never succeed.</td>
	</tr>
	<tr>
		<td class="text-right">RETRY</td>
		<td>Request is valid and understood, but can't be processed at this time. This request can be retried.</td>
	</tr>
	<tr>
		<td class="text-right">RETRY_LATER</td>
		<td>This request can be retried later after certain state/ error condition has been changed.</td>
	</tr>
	<tr>
		<td class="text-right">OTHER_MEANS</td>
		<td>Special case of retry. Please provide another means of payment.</td>
	</tr>
</table>


List of error names (these names will not change, so you may parse these and attach your logic to the ErrorName):

<table class="table table-striped">
	<tr>
		<td class="text-right col-sm-4">AUTHENTICATION_FAILED</td>
		<td class="col-sm-8">
			Wrong password, wrong client certificate, invalid token, wrong HMAC.<br />
			<i>Solution:</i> Use proper credentials, fix HMAC calculation, use valid token
		</td>
	</tr>
	<tr>
		<td class="text-right">ALIAS_INVALID</td>
		<td>
			The alias is not known or already used (in case of registration).<br />
			<i>Solution:</i> Use another alias for registration
		</td>
	</tr>
	<tr>
		<td class="text-right">BLOCKED_BY_RISK_MANAGEMENT</td>
		<td>
			Action blocked by risk management<br />
			<i>Solution:</i> Unblock in Saferpay Risk Management (Backoffice)
		</td>
	</tr>
	<tr>
		<td class="text-right">CARD_CHECK_FAILED</td>
		<td>
			Invalid card number or cvc (this is only returned for the SIX-internal chard check feature for Alias/InsertDirect).<br />
			<i>Solution:</i> Let the card holder correct the entered data
		</td>
	</tr>
	<tr>
		<td class="text-right">CARD_CVC_INVALID</td>
		<td>
			Wrong cvc entered<br />
			<i>Solution:</i> Retry with correct cvc
		</td>
	</tr>
	<tr>
		<td class="text-right">CARD_CVC_REQUIRED</td>
		<td>
			Cvc not entered but required<br />
			<i>Solution:</i> Retry with cvc entered
		</td>
	</tr>
	<tr>
		<td class="text-right">CARD_EXPIRED</td>
		<td>
			Card expired<br />
			<i>Solution:</i> Use another card or correct expiry date
		</td>
	</tr>
	<tr>
		<td class="text-right">PAYMENTMEANS_INVALID</td>
		<td>
			Invalid means of payment (e.g. invalid card)
		</td>
	</tr>
	<tr>
		<td class="text-right">INTERNAL_ERROR</td>
		<td>
			Internal error in Saferpay<br />
			<i>Solution:</i> Try again
		</td>
	</tr>
	<tr>
		<td class="text-right">3DS_AUTHENTICATION_FAILED</td>
		<td>
			3D-secure authentication failed – the transaction must be aborted.<br />
			<i>Solution:</i> Use another card or means of payment
		</td>
	</tr>
	<tr>
		<td class="text-right">NO_CONTRACT</td>
		<td>
			No contract available for the brand / currency combination.<br />
			<i>Solution:</i> Use another card or change the currency to match an existing contract or have the currency activated for the contract.
		</td>
	</tr>
	<tr>
		<td class="text-right">NO_CREDITS_AVAILABLE</td>
		<td>
			No more credits available for this account.<br />
			<i>Solution:</i> Buy more transaction credits
		</td>
	</tr>
	<tr>
		<td class="text-right">PERMISSION_DENIED</td>
		<td>
			No permission (e.g. terminal does not belong to the customer)
		</td>
	</tr>
	<tr>
		<td class="text-right">TRANSACTION_DECLINED</td>
		<td>
			Declined by the processor.<br />
			<i>Solution:</i> Use another card or check details.
		</td>
	</tr>
	<tr>
		<td class="text-right">VALIDATION_FAILED</td>
		<td>
			Validation failed.<br />
			<i>Solution:</i> Fix request
		</td>
	</tr>
	<tr>
		<td class="text-right">AMOUNT_INVALID</td>
		<td>
			The amount does not adhere to the restrictions for this action. E.g. it might be exceeding the allowed capture amount.
		</td>
	</tr>
	<tr>
		<td class="text-right">CURRENCY_INVALID</td>
		<td>
			Currency does not match referenced transaction currency.
		</td>
	</tr>
	<tr>
		<td class="text-right">COMMUNICATION_FAILED</td>
		<td>
			The communication to the processor failed.<br />
			<i>Solution:</i> Try again or use another means of payment
		</td>
	</tr>
	<tr>
		<td class="text-right">COMMUNICATION_TIMEOUT</td>
		<td>
			Saferpay did not receive a response from the external system in time. It’s possible that an authorization was created, but Saferpay is not able to know this.<br />
			<i>Solution:</i> Check with the acquirer if there is an authorization which needs to be canceled.
		</td>
	</tr>
	<tr>
		<td class="text-right">TOKEN_EXPIRED</td>
		<td>
			The token is expired.<br />
			<i>Solution:</i> Create a new token.
		</td>
	</tr>
	<tr>
		<td class="text-right">TOKEN_INVALID</td>
		<td>
			The token either does not exist for this customer or was already used
		</td>
	</tr>
	<tr>
		<td class="text-right">TRANSACTION_IN_WRONG_STATE</td>
		<td></td>
	</tr>
	<tr>
		<td class="text-right">ACTION_NOT_SUPPORTED</td>
		<td></td>
	</tr>
	<tr>
		<td class="text-right">TRANSACTION_NOT_FOUND</td>
		<td></td>
	</tr>
	<tr>
		<td class="text-right">CONDITION_NOT_SATISFIED</td>
		<td>
			The condition which was defined in the request could not be satisfied.
		</td>
	</tr>
	<tr>
		<td class="text-right">TRANSACTION_NOT_STARTED</td>
		<td>
			The transaction was not started by the payer. Therefore, no final result for the transaction is available.<br />
			<i>Solution:</i> Try again later.
		</td>
	</tr>
	<tr>
		<td class="text-right">TRANSACTION_ABORTED</td>
		<td>
			The transaction was aborted by the payer.
		</td>
	</tr>
</table>

<<<---

# <a name="ChapterPaymentPage"></a>Payment Page



## <a name="Payment_v1_PaymentPage_Initialize"></a>PaymentPage Initialize

This interface provides a simple integration of Saferpay without the need to implement a user interface for card data entry (the 'eCommerce' Saferpay license). You will get a redirect link to our payment page.

After the payment page processing was finished, the payer is redirected back to the shop. The redirect address is chosen depending on the outcome of the request (success, failure, abort). If the payer is returned to the success url provided in the InitializeRequest, an authorization or even a completed transaction will have been done. So even if you don’t call Verify or Capture, the financial flow may have been triggered (depending on the payment provider – please consult the provider specific information).

Important: the payer might modify the return address (e.g. replace failure address with success url). If the payer returns to your success url, be sure to retrieve the outcome of the transaction and more information on it by calling PaymentPage/Verify Assert for the given token.

--->>>

```
POST /Payment/v1/PaymentPage/Initialize
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ConfigSet</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">This parameter let you define your payment page config (PPConfig) by name. If this parameters is not set, your default PPConfig will be applied if available.
 When the PPConfig can't be found (e.g. wrong name), the Saferpay basic style will be applied to the payment page.</div>
	<i class="small text-muted">
Id[1..20]<br />
					<span>Example: name of your payment page config (case-insensitive)</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay terminal id</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12345678</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payment</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentPagePayment">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payment (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMethods</strong><br />
	<span class="text-muted small">
		string[]
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to restrict the means of payment which are available to the payer for this transaction. If only one payment method id is set, the payment selection step will be skipped.</div>
	<i class="small text-muted">
Possible values: AMEX, BANCONTACT, BONUS, DINERS, DIRECTDEBIT, EPRZELEWY, EPS, GIROPAY, IDEAL, INVOICE, JCB, MAESTRO, MASTERCARD, MYONE, PAYPAL, POSTCARD, POSTFINANCE, SAFERPAYTEST, SOFORT, VISA, VPAY.<br />
					<span>Example: [&quot;VISA&quot;, &quot;MASTERCARD&quot;]</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Wallets</strong><br />
	<span class="text-muted small">
		string[]
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to control if wallets should be enabled on the payment selection page and to go directly to the given wallet (if exactly one wallet is filled and PaymentMethods is not set).</div>
	<i class="small text-muted">
Possible values: MASTERPASS.<br />
					<span>Example: [&quot;MASTERPASS&quot;]</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Payer">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegisterAlias</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegisterAlias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">If the given means of payment should be stored in Saferpay Secure Card Data storage (if applicable)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ReturnUrls</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_ReturnUrls">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Urls which are to be used to redirect the payer back to the shop if the transaction requires some kind of browser redirection (3d-secure, dcc)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Notification</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Notification">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Notification options</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Styling</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Styling">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Styling options</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>BillingAddressForm</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_AddressForm">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to have the payer enter his address in the payment process. Only one address form is supported at this time!</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>DeliveryAddressForm</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_AddressForm">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to have the payer enter his address in the payment process. Only one address form is supported at this time!</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>CardForm</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_CardForm">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Options for card data entry form (if applicable)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request identifier]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Payment": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "OrderId": "Id of the order",
    "Description": "Description of payment"
  },
  "ReturnUrls": {
    "Success": "[your shop payment success url]",
    "Fail": "[your shop payment fail url]"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token for later referencing</div>
	<i class="small text-muted">
					<span>Example: 234uhfh78234hlasdfh8234e1234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Expiration</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Expiration date / time of the generated token in ISO 8601 format in UTC. After this time, the token won’t be accepted for any further action.</div>
	<i class="small text-muted">
					<span>Example: 2011-07-14T19:43:37+01:00</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectUrl</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Redirecturl for the payment page transaction. Simply add this to a "Pay Now"-button or do an automatic redirect.</div>
	<i class="small text-muted">
					<span>Example: https://www.saferpay.com/vt2/api/PaymentPage/1234/12341234/z2p7a0plpgsd41m97wjvm5jza</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "Id of the request"
  },
  "Token": "234uhfh78234hlasdfh8234e1234",
  "Expiration": "2015-01-30T12:45:22.258+01:00",
  "RedirectUrl": "https://www.saferpay.com/vt2/api/..."
}
</pre>

<<<---





## <a name="Payment_v1_PaymentPage_Assert"></a>PaymentPage Assert

Call this function to safely check the status of the transaction from your server. Depending on the payment provider, the resulting transaction may either be an authorization or may already be captured (meaning the financial flow was already triggered). This will be visible in the status of the transaction container returned in the response.

If the transaction failed (the payer was redirected to the Fail url or he manipulated the return url), an error response with an http status code 400 or higher containing an error message will be returned providing some information on the transaction failure.

--->>>

```
POST /Payment/v1/PaymentPage/Assert
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by initial call.</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request identifier]",
    "RetryIndicator": 0
  },
  "Token": "234uhfh78234hlasdfh8234e"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegistrationResult</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegistrationResult">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the SCD registration outcome</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ThreeDs</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_ThreeDsInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">3d-secure information if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Dcc</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DccInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Dcc information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "PAYMENT",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "Saferpay Test Card",
    "AcquirerReference": "000000"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---








# <a name="ChapterTransaction"></a>Transaction



## <a name="Payment_v1_Transaction_Initialize"></a>Transaction Initialize <span class="label text-mandatory">Business license</span> 

This method may be used to start a transaction which may involve either DCC and / or 3d-secure.

--->>>

```
POST /Payment/v1/Transaction/Initialize
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ConfigSet</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">This parameter let you define your payment page config (PPConfig) by name. If this parameters is not set, your default PPConfig will be applied if available.
 When the PPConfig can't be found (e.g. wrong name), the Saferpay basic style will be applied to the payment page.</div>
	<i class="small text-muted">
Id[1..20]<br />
					<span>Example: name of your payment page config (case-insensitive)</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay terminal to use for this authorization</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payment</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Payment">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payment (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeans">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Means of payment (either card data or a reference to a previously stored card).
 Important: Only fully PCI certified merchants may directly use the card data. If your system is not explicitly certified to handle card data directly, then use the Saferpay Secure Card Data-Storage instead. If the customer enters a new card, you may want to use the Saferpay Hosted Register Form to capture the card data through Saferpay.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Payer">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the payer (IP-address)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ReturnUrls</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_ReturnUrls">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Urls which are to be used to redirect the payer back to the shop if the transaction requires some kind of browser redirection (3d-secure, dcc)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Styling</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Styling">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Styling options</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Wallet</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Wallet">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Wallet system to be used for the transaction (this cannot be combined with PaymentMeans above).</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Payment": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    }
  },
  "Payer": {
    "LanguageCode": "en"
  },
  "ReturnUrls": {
    "Success": "[your shop payment success url]",
    "Fail": "[your shop payment fail url]"
  },
  "Styling": {
    "CssUrl": "[your shop css url]"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id for referencing later</div>
	<i class="small text-muted">
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Expiration</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Expiration date / time of the generated token in ISO 8601 format in UTC. After this time, the token won’t be accepted for any further action.</div>
	<i class="small text-muted">
					<span>Example: 2015-01-30T13:45:22.258+02:00</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>LiabilityShift</strong><br />
	<span class="text-muted small">
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Indicates if liability shift to issuer is possible or not. Not present if PaymentMeans container was not present in InitializeTransaction request. True, if liability shift to issuer is possible, false if not.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectRequired</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">True if a redirect must be performed to continue, false otherwise</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Redirect</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Redirect">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Mandatory if RedirectRequired is true. Contains the URL for the redirect to use for example the Saferpay hosted register form.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Token": "234uhfh78234hlasdfh8234e",
  "Expiration": "2015-01-30T12:45:22.258+01:00",
  "LiabilityShift": false,
  "RedirectRequired": true,
  "Redirect": {
    "RedirectUrl": "https://www.saferpay.com/vt2/Api/...",
    "PaymentMeansRequired": true
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_Authorize"></a>Transaction Authorize <span class="label text-mandatory">Business license</span> 

This function may be called to complete an authorization which was started by a call to Transaction/Initialize.

--->>>

```
POST /Payment/v1/Transaction/Authorize
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by Initialize</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Condition</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">WITH_LIABILITY_SHIFT: the authorization will be executed if the previous 3d-secure process indicates that the liability shift to the issuer is possible (liability shift may still be declined with the authorization though). This condition will be ignored for brands which Saferpay does not offer 3d-secure for.

If left out, the authorization will be done if allowed, but possibly without liability shift to the issuer. See the specific result codes in the response message.</div>
	<i class="small text-muted">
Possible values: WITH_LIABILITY_SHIFT.<br />
					<span>Example: WITH_LIABILITY_SHIFT</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>VerificationCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Card verification code if available</div>
	<i class="small text-muted">
Numeric[3..4]<br />
					<span>Example: 123</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegisterAlias</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegisterAlias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Controls whether the given means of payment should be stored inside the saferpay Secure Card Data storage.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "Token": "sdu5ymxx210y2dz1ggig2ey0o",
  "VerificationCode": "123"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegistrationResult</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegistrationResult">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the Secure Card Data registration outcome.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ThreeDs</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_ThreeDsInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">3d-secure information if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Dcc</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DccInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Dcc information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "PAYMENT",
    "Status": "AUTHORIZED",
    "Id": "MUOGAWA9pKr6rAv5dUKIbAjrCGYA",
    "Date": "2015-09-18T09:19:27.078Z",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "AcquirerName",
    "AcquirerReference": "Reference"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "SaferpayTestCard"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  },
  "Payer": {
    "IpAddress": "1.2.3.4",
    "IpLocation": "DE"
  },
  "ThreeDs": {
    "Authenticated": true,
    "LiabilityShift": true,
    "Xid": "ARkvCgk5Y1t/BDFFXkUPGX9DUgs=",
    "VerificationValue": "AAABBIIFmAAAAAAAAAAAAAAAAAA="
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_QueryPaymentMeans"></a>Transaction QueryPaymentMeans <span class="label text-mandatory">Business license</span> 

This method may be used to query the payment means and payer data (address) after initialize and wallet redirect.

--->>>

```
POST /Payment/v1/Transaction/QueryPaymentMeans
```

<<<---

#### Request


This function may be called to retrieve information on the means of payment which was entered / chosen by the payer after the browser is redirected to the successUrl.
 For MasterPass, the address the payer has selected in his wallet is returned as well as the RedirectUrl for DCC if DCC was skipped by the EnableAmountAdjustment attribute in Initialize.

<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by Initialize</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ReturnUrls</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_ReturnUrls">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Urls which are to be used to redirect the payer back to the shop if the transaction requires some kind of browser redirection (3d-secure, dcc)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "Token": "sdu5ymxx210y2dz1ggig2ey0o"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectRequired</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px"></div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectUrl</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Available if DCC may be performed.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "SaferpayTestCard"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  },
  "Payer": {
    "IpAddress": "1.2.3.4",
    "IpLocation": "DE"
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_AdjustAmount"></a>Transaction AdjustAmount <span class="label text-mandatory">Business license</span> 

This method may be used to adjust the amount after query payment means.

--->>>

```
POST /Payment/v1/Transaction/AdjustAmount
```

<<<---

#### Request


This function allows a change of the authorization amount which was originally set by Initialize.
 For the time being, this is only allowed for MasterPass business integration scenario and requires a flag having been set in the Initialize call.

<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by Initialize</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The new amount</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "Token": "sdu5ymxx210y2dz1ggig2ey0o",
  "Amount": {
    "Value": "110",
    "CurrencyCode": "CHF"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_AuthorizeDirect"></a>Transaction AuthorizeDirect <span class="label text-mandatory">Business license</span> 

This function may be used to directly authorize transactions which do not require a redirect of the customer (e.g. direct debit or recurring transactions based on a previously registered alias).

--->>>

```
POST /Payment/v1/Transaction/AuthorizeDirect
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay Terminal-Id</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payment</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Payment">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payment (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeans">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegisterAlias</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegisterAlias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Controls whether the given means of payment should be stored inside the saferpay Secure Card Data storage.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Payer">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the payer (IP-address)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Payment": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "Description": "Test123",
    "PayerNote": "Order123_Testshop"
  },
  "PaymentMeans": {
    "Card": {
      "Number": "912345678901234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "VerifivationCode": "123"
    }
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegistrationResult</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegistrationResult">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the Secure Card Data registration outcome.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "PAYMENT",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "AcquirerName",
    "AcquirerReference": "Reference"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "SaferpayTestCard"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 7,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  },
  "Payer": {
    "IpAddress": "1.2.3.4",
    "IpLocation": "DE"
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_AuthorizeReferenced"></a>Transaction AuthorizeReferenced <span class="label text-mandatory">Business license</span> 

This method may be used to perform follow-up authorizations to an earlier transaction. At this time, the referenced (initial) transaction must have been performed setting either the recurring or installment option.

--->>>

```
POST /Payment/v1/Transaction/AuthorizeReferenced
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay Terminal-Id</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payment</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_BasicPayment">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payment (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionReference</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_TransactionReference">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>SuppressDcc</strong><br />
	<span class="text-muted small">
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to suppress direct currency conversion</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Payment": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "Description": "Test123",
    "PayerNote": "Order123_Testshop"
  },
  "TransactionReference": {
    "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Dcc</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DccInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Dcc information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "PAYMENT",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "AcquirerName",
    "AcquirerReference": "Reference"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "SaferpayTestCard"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 7,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  },
  "Payer": {
    "IpAddress": "1.2.3.4",
    "IpLocation": "DE"
  },
  "Dcc": {
    "PayerAmount": {
      "Value": "109",
      "CurrencyCode": "USD"
    }
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_Capture"></a>Transaction Capture



--->>>

```
POST /Payment/v1/Transaction/Capture
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionReference</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_TransactionReference">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Reference to authorization.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Currency must match the original transaction currency (request will be declined if currency does not match)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Billpay</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_BillpayCapture">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Optional Billpay specific options.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Partial</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PartialCapture">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Optional partial capture options.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TransactionReference": {
    "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id of the referenced transaction.</div>
	<i class="small text-muted">
AlphaNumeric[1..64]<br />
					<span>Example: 723n4MAjMdhjSAhAKEUdA8jtl9jb</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">OrderId of the referenced transaction. If present.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Date</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Date and time of capture.</div>
	<i class="small text-muted">
AlphaNumeric[11..11]<br />
					<span>Example: 2014-04-25T08:33:44.159Z</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Invoice</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_InvoiceInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Optional infos for invoice based payments.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
  "Date": "2015-01-30T12:45:22.258+01:00"
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_Refund"></a>Transaction Refund <span class="label text-mandatory">Business license</span> 

This method may be called to refund a previous transaction.

--->>>

```
POST /Payment/v1/Transaction/Refund
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Refund</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Refund">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the refund (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionReference</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_CaptureTransactionReference">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Reference to the transaction you want to refund. Note, that not all processors support refunds.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[your request id]",
    "RetryIndicator": 0
  },
  "Refund": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    }
  },
  "TransactionReference": {
    "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Dcc</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DccInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Dcc information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "REFUND",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "Saferpay Test",
    "AcquirerReference": "000000"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_RefundDirect"></a>Transaction RefundDirect <span class="label text-mandatory">Business license</span> 

This method may be called to refund an amount to the given means of payment (not supported for all means of payment) without referencing a previous transaction. This might be the case if the original transaction was done with a card which is not valid any more.

--->>>

```
POST /Payment/v1/Transaction/RefundDirect
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay Terminal-Id</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Refund</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Refund">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the refund (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeans">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[your request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Refund": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    }
  },
  "PaymentMeans": {
    "Alias": {
      "Id": "alias35nfd9mkzfw0x57iwx"
    }
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Dcc</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DccInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Dcc information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "REFUND",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "Saferpay Test",
    "AcquirerReference": "000000"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_Cancel"></a>Transaction Cancel



--->>>

```
POST /Payment/v1/Transaction/Cancel
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionReference</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_TransactionReference">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Reference to transaction to be canceled.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TransactionReference": {
    "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TransactionId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id of the referenced transaction.</div>
	<i class="small text-muted">
					<span>Example: qiuwerhfi23h4189asdhflk23489</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">OrderId of the referenced transaction. If present.</div>
	<i class="small text-muted">
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Date</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Date and time of cancel.</div>
	<i class="small text-muted">
					<span>Example: 2014-04-25T08:33:44.159Z</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "TransactionId": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
  "OrderId": "c52ad18472354511ab2c33b59e796901",
  "Date": "2015-01-30T12:45:22.258+01:00"
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_RedirectPayment"></a>Transaction RedirectPayment <span class="label text-mandatory">Business license</span> 



--->>>

```
POST /Payment/v1/Transaction/RedirectPayment
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay terminal to use for this authorization</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payment</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Payment">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payment (amount, currency, ...)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ServiceProvider</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Service provider to be used for this payment</div>
	<i class="small text-muted">
Possible values: PAYPAL, POSTCARD, POSTFINANCE.<br />
					<span>Example: PAYPAL</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Payer">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the payer (IP-address)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ReturnUrls</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_ReturnUrls">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Urls which are to be used to redirect the payer back to the shop if the transaction requires some kind of browser redirection (3d-secure, dcc)</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Styling</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Styling">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Custom styling resource</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Notification</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Notification">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Notification options</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[your request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]",
  "Payment": {
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    }
  },
  "ServiceProvider": "PAYPAL",
  "ReturnUrls": {
    "Success": "[your shop payment success url]",
    "Fail": "[your shop payment fail url]"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id for referencing later</div>
	<i class="small text-muted">
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Expiration</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Expiration date / time of the generated token in ISO 8601 format in UTC. After this time, the token won’t be accepted for any further action.</div>
	<i class="small text-muted">
					<span>Example: 2015-01-30T13:45:22.258+02:00</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectUrl</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Url to redirect the browser to for payment processing</div>
	<i class="small text-muted">
					<span>Example: https://www.saferpay.com/VT2/api/...</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Token": "234uhfh78234hlasdfh8234e",
  "Expiration": "2015-01-30T12:45:22.258+01:00",
  "RedirectUrl": "https://www.saferpay.com/vt2/Api/..."
}
</pre>

<<<---





## <a name="Payment_v1_Transaction_AssertRedirectPayment"></a>Transaction AssertRedirectPayment <span class="label text-mandatory">Business license</span> 



--->>>

```
POST /Payment/v1/Transaction/AssertRedirectPayment
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by initial call.</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request identifier]",
    "RetryIndicator": 0
  },
  "Token": "234uhfh78234hlasdfh8234e"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Transaction</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Transaction">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the transaction</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Payer</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PayerInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payer / card holder</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Transaction": {
    "Type": "PAYMENT",
    "Status": "AUTHORIZED",
    "Id": "723n4MAjMdhjSAhAKEUdA8jtl9jb",
    "Date": "2015-01-30T12:45:22.258+01:00",
    "Amount": {
      "Value": "100",
      "CurrencyCode": "CHF"
    },
    "AcquirerName": "Saferpay Test",
    "AcquirerReference": "8EZRQVT0ODW4ME525"
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "Number": "912345678901234",
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---








# <a name="ChapterAliasStore"></a>Secure Alias Store



## <a name="Payment_v1_Alias_Insert"></a>Alias Insert

This function may be used to insert an alias without knowledge about the card details. Therefore a redirect of the customer is required.

--->>>

```
POST /Payment/v1/Alias/Insert
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegisterAlias</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_RegisterAlias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Registration parameters</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Type</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Type of payment means to register</div>
	<i class="small text-muted">
Possible values: CARD, BANK_ACCOUNT, POSTFINANCE.<br />
					<span>Example: CARD</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ReturnUrls</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_ReturnUrls">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Urls which are to be used to redirect the payer back to the shop.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Styling</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Styling">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Custom styling resource for the Hosted Register form.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>LanguageCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Language used for displaying forms.

Code-List:
 de - German
 en - English
 fr - French
 da - Danish
 cs - Czech
 es - Spanish
 hr - Croatian
 it - Italian
 hu - Hungarian
 nl - Dutch
 no - Norwegian
 pl - Polish
 pt - Portuguese
 ru - Russian
 ro - Romanian
 sk - Slovak
sl - Slovenian
 fi - Finnish
sv - Swedish
 tr - Turkish
 el - Greek
 ja - Japanese
 zh - Chinese</div>
	<i class="small text-muted">
					<span>Example: de</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Check</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_AliasInsertCheck">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Parameters for checking the means of payment before registering.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[your request id]",
    "RetryIndicator": 0
  },
  "RegisterAlias": {
    "IdGenerator": "RANDOM"
  },
  "Type": "CARD",
  "ReturnUrls": {
    "Success": "[your shop alias registration success url]",
    "Fail": "[your shop alias registration fail url]"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id for referencing later</div>
	<i class="small text-muted">
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Expiration</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Expiration date / time of the generated token in ISO 8601 format in UTC. After this time, the token won’t be accepted for any further action.</div>
	<i class="small text-muted">
					<span>Example: 2015-01-30T13:45:22.258+02:00</span>
	</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RedirectUrl</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay-Url to post the form data to.</div>
	<i class="small text-muted">
					<span>Example: https://www.saferpay.com/VT2/api/...</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Token": "234uhfh78234hlasdfh8234e",
  "Expiration": "2015-01-30T12:45:22.258+01:00",
  "RedirectUrl": "https://www.saferpay.com/vt2/api/..."
}
</pre>

<<<---





## <a name="Payment_v1_Alias_AssertInsert"></a>Alias AssertInsert



--->>>

```
POST /Payment/v1/Alias/AssertInsert
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Token</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Token returned by initial call.</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 234uhfh78234hlasdfh8234e</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request identifier]",
    "RetryIndicator": 0
  },
  "Token": "234uhfh78234hlasdfh8234e"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Alias</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_AliasInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the registered alias.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the registered means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Alias": {
    "Id": "alias35nfd9mkzfw0x57iwx",
    "Lifetime": 1000
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---





## <a name="Payment_v1_Alias_InsertDirect"></a>Alias InsertDirect



--->>>

```
POST /Payment/v1/Alias/InsertDirect
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RegisterAlias</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_RegisterAlias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Registration parameters</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeans">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Means of payment to register</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[your request id]",
    "RetryIndicator": 0
  },
  "PaymentMeans": {
    "Card": {
      "Number": "912345678901234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "VerificationCode": "123"
    }
  },
  "RegisterAlias": {
    "IdGenerator": "RANDOM"
  }
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>Alias</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_AliasInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the registered alias.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>PaymentMeans</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentMeansInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the registered means of payment</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  },
  "Alias": {
    "Id": "alias35nfd9mkzfw0x57iwx",
    "Lifetime": 1000
  },
  "PaymentMeans": {
    "Brand": {
      "PaymentMethod": "SAFERPAYTEST",
      "Name": "Saferpay Test Card"
    },
    "DisplayText": "9123 45xx xxxx 1234",
    "Card": {
      "MaskedNumber": "912345xxxxxx1234",
      "ExpYear": 2015,
      "ExpMonth": 9,
      "HolderName": "Max Mustermann",
      "CountryCode": "CH"
    }
  }
}
</pre>

<<<---





## <a name="Payment_v1_Alias_Delete"></a>Alias Delete



--->>>

```
POST /Payment/v1/Alias/Delete
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>AliasId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The Alias you want to delete</div>
	<i class="small text-muted">
Id[1..40]<br />
					<span>Example: alias35nfd9mkzfw0x57iwx</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "AliasId": "alias35nfd9mkzfw0x57iwx"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  }
}
</pre>

<<<---








# <a name="ChapterBatch"></a>Batch



## <a name="Payment_v1_Batch_Close"></a>Batch Close

No documentation available

--->>>

```
POST /Payment/v1/Batch/Close
```

<<<---

#### Request




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>RequestHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_RequestHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">General information about the request.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay Terminal-Id</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "RequestHeader": {
    "SpecVersion": "1.3",
    "CustomerId": "[your customer id]",
    "RequestId": "[unique request id]",
    "RetryIndicator": 0
  },
  "TerminalId": "[your terminal id]"
}
</pre>

<<<---

#### Response




<table class="table">
	<thead>
		<tr>
			<th colspan="2">Arguments</th>
		</tr>
	</thead>
				<tr>
					<td class="col-sm-4 text-right">
	<strong>ResponseHeader</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Common_ResponseHeader">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Contains general informations about the response.</div>
	<i class="small text-muted">
			</i>
</td>
				</tr>

</table>


--->>>

<p>Example:</p>
<pre class="prettyprint">
{
  "ResponseHeader": {
    "SpecVersion": "1.3",
    "RequestId": "[your request id]"
  }
}
</pre>

<<<---








<div class="visible-print-block" id="type-dict">
	<h4>Container Dictionary</h4>
			<h2>Container "Common_RequestHeader"</h2>
				<table class="table" id="Common_RequestHeader">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>SpecVersion</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Version number of the interface specification. For new implementations, the newest Version should be used.</div>
	<i class="small text-muted">
Possible values: 1.0, 1.1, 1.2, 1.3, 1.4.<br />
					<span>Example: 1.0</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CustomerId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay customer id. Part of the Saferpay AccountID, which has the following format: 123123-12345678. The first Value is your CustomerID.</div>
	<i class="small text-muted">
Numeric[1..8]<br />
					<span>Example: 123123</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>RequestId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unique id generated by merchant. Must not change for subsequently resent requests.</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 33e8af17-35c1-4165-a343-c1c86a320f3b</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>RetryIndicator</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">0 if the request is sent for the first time, >=1 if it is a repetition.</div>
	<i class="small text-muted">
Range: inclusive between 0 and 9<br />
					<span>Example: 0</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ClientInfo</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_ClientInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the caller (merchant host)</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Common_ResponseHeader"</h2>
				<table class="table" id="Common_ResponseHeader">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>RequestId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">RequestId of the original request header</div>
	<i class="small text-muted">
Id[1..50]<br />
					<span>Example: 33e8af17-35c1-4165-a343-c1c86a320f3b</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>SpecVersion</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Version number of the interface specification.</div>
	<i class="small text-muted">
Possible values: 1.0, 1.1, 1.2.<br />
					<span>Example: 1.0</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_BankAccountInfo"</h2>
				<table class="table" id="Payment_Models_BankAccountInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IBAN</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">International Bank Account Number in electronical format (without spaces).</div>
	<i class="small text-muted">
AlphaNumeric[1..50]<br />
					<span>Example: DE12500105170648489890</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HolderName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the account holder</div>
	<i class="small text-muted">
Iso885915[1..50]<br />
					<span>Example: John Doe</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BIC</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Bank Identifier Code without spaces.</div>
	<i class="small text-muted">
AlphaNumeric[8..11]<br />
					<span>Example: INGDDEFFXXX</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BankName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the Bank</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CountryCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">ISO 2-letter country code of the Iban origin (if available)</div>
	<i class="small text-muted">
					<span>Example: CH</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Address"</h2>
				<table class="table" id="Payment_Models_Data_Address">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>FirstName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers first name</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: John</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>LastName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers last name</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: Doe</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DateOfBirth</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers date of birth in ISO 8601 extended date notation
 YYYY-MM-DD</div>
	<i class="small text-muted">
AlphaNumeric[11..11]<br />
					<span>Example: 1990-05-31</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Company</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers company</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: ACME Corp.</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Gender</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers gender</div>
	<i class="small text-muted">
Possible values: MALE, FEMALE, COMPANY.<br />
					<span>Example: COMPANY</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>LegalForm</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers legal form</div>
	<i class="small text-muted">
Possible values: AG, GmbH, Misc.<br />
					<span>Example: AG</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Street</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers street</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: Bakerstreet 32</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Street2</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers street, second line. Only use this, if you need two lines. It may not be supported by all acquirers.</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: Stewart House</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Zip</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers zip code</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: 8000</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>City</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers city</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: Zurich</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CountrySubdivisionCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers country subdivision code
 ISO 3166-2 country subdivision code</div>
	<i class="small text-muted">
Alphabetic[1..3]<br />
					<span>Example: ZH</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CountryCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers country subdivision code
 ISO 3166-1 alpha-2 country code</div>
	<i class="small text-muted">
Alphabetic[2..2]<br />
					<span>Example: CH</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Phone</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers phone number</div>
	<i class="small text-muted">
Utf8[1..100]<br />
					<span>Example: +41 12 345 6789</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Email</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The payers email</div>
	<i class="small text-muted">
					<span>Example: payer@provider.com</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_AddressForm"</h2>
				<table class="table" id="Payment_Models_Data_AddressForm">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Display</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Specifes whether to show address form or not.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MandatoryFields</strong><br />
	<span class="text-muted small">
		string[]
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">List of fields which the payer must enter to proceed with the payment.</div>
	<i class="small text-muted">
Possible values: CITY, COMPANY, COUNTRY, EMAIL, FIRSTNAME, LASTNAME, PHONE, SALUTATION, STATE, STREET, ZIP.<br />
					<span>Example: [&quot;FIRSTNAME&quot;, &quot;LASTNAME&quot;, &quot;PHONE&quot;]</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Alias"</h2>
				<table class="table" id="Payment_Models_Data_Alias">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Id</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id / name of the alias.</div>
	<i class="small text-muted">
Id[1..40]<br />
					<span>Example: alias35nfd9mkzfw0x57iwx</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>VerificationCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Verification code (CVV, CVC) if applicable (ie the alias referenced is a card).</div>
	<i class="small text-muted">
Numeric[3..4]<br />
					<span>Example: 123</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_AliasInfo"</h2>
				<table class="table" id="Payment_Models_Data_AliasInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Id</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id / name of alias</div>
	<i class="small text-muted">
					<span>Example: alias35nfd9mkzfw0x57iwx</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Lifetime</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Number of days this card is stored within Saferpay. Minimum 1 day, maximum 1600 days.</div>
	<i class="small text-muted">
					<span>Example: 1000</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_AliasInsertCheck"</h2>
				<table class="table" id="Payment_Models_Data_AliasInsertCheck">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Type</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Type of check to perform.</div>
	<i class="small text-muted">
Possible values: ONLINE.<br />
					<span>Example: ONLINE</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>TerminalId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Saferpay Terminal-Id to be user for checking.</div>
	<i class="small text-muted">
Numeric[8..8]<br />
					<span>Example: 12341234</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Amount"</h2>
				<table class="table" id="Payment_Models_Data_Amount">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Value</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount in minor unit (CHF 1.00 &rArr; Value=100)</div>
	<i class="small text-muted">
					<span>Example: 100</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CurrencyCode</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">ISO 4217 3-letter currency code (CHF, USD, EUR, ...)</div>
	<i class="small text-muted">
					<span>Example: CHF</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_BankAccount"</h2>
				<table class="table" id="Payment_Models_Data_BankAccount">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IBAN</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">International Bank Account Number in electronical format (without spaces).</div>
	<i class="small text-muted">
AlphaNumeric[1..50]<br />
					<span>Example: DE12500105170648489890</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HolderName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the account holder</div>
	<i class="small text-muted">
Iso885915[1..50]<br />
					<span>Example: John Doe</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BIC</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Bank Identifier Code without spaces.</div>
	<i class="small text-muted">
AlphaNumeric[8..11]<br />
					<span>Example: INGDDEFFXXX</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BankName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the Bank</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_BasicPayment"</h2>
				<table class="table" id="Payment_Models_Data_BasicPayment">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount data (currency, value, etc.)</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unambiguous order identifier defined by the merchant/ shop. This identifier might be used as reference later on.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Description</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">A human readable description provided by the merchant that can be displayed in web sites.</div>
	<i class="small text-muted">
Utf8[1..1000]<br />
					<span>Example: Description</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PayerNote</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Text which will be printed on payere's debit note. Supported by SIX Acquiring. No guarantee, that it will show up on the payer's debit note, because his bank has to support it too.
 Please note, that maximum allowed characters are rarely supported. It's usually around 10-12.</div>
	<i class="small text-muted">
Utf8[1..50]<br />
					<span>Example: Payernote</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_BillpayCapture"</h2>
				<table class="table" id="Payment_Models_Data_BillpayCapture">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DelayInDays</strong><br />
	<span class="text-muted small">
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Number of days to delay the due date of the invoice / direct debit (see billpay for specifics)</div>
	<i class="small text-muted">
					<span>Example: alias35nfd9mkzfw0x57iwx</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Brand"</h2>
				<table class="table" id="Payment_Models_Data_Brand">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PaymentMethod</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">alphanumeric id of the payment method / brand</div>
	<i class="small text-muted">
Possible values: AMEX, BANCONTACT, BONUS, DINERS, DIRECTDEBIT, EPRZELEWY, EPS, GIROPAY, IDEAL, INVOICE, JCB, MAESTRO, MASTERCARD, MYONE, PAYPAL, POSTCARD, POSTFINANCE, SAFERPAYTEST, SOFORT, VISA, VPAY.<br />
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Name</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the Brand (Visa, Mastercard, an so on – might change over time, only use for display, never for parsing). Only use it for display, never for parsing and/or mapping! Use PaymentMethod instead.</div>
	<i class="small text-muted">
					<span>Example: SaferpayTestCard</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_CaptureTransactionReference"</h2>
				<table class="table" id="Payment_Models_Data_CaptureTransactionReference">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>TransactionId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id of the referenced transaction.</div>
	<i class="small text-muted">
AlphaNumeric[1..64]<br />
					<span>Example: 723n4MAjMdhjSAhAKEUdA8jtl9jb</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unambiguous OrderId of the referenced transaction.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderPartId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">OrderPartId of the reference transaction if a partial capture should be referenced and the TransactionId of the partial capture is not available.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: kh9ngajrfe6wfu3d0c</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Card"</h2>
				<table class="table" id="Payment_Models_Data_Card">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Number</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Card number without separators</div>
	<i class="small text-muted">
Numeric[1..50]<br />
					<span>Example: 1234123412341234</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ExpYear</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Year of expiration</div>
	<i class="small text-muted">
Range: inclusive between 2000 and 9999<br />
					<span>Example: 2015</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ExpMonth</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Month of expiration (eg 9 for September)</div>
	<i class="small text-muted">
Range: inclusive between 1 and 12<br />
					<span>Example: 9</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HolderName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the card holder</div>
	<i class="small text-muted">
Utf8[1..50]<br />
					<span>Example: John Doe</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>VerificationCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Verification code (CVV, CVC) if applicable</div>
	<i class="small text-muted">
Numeric[3..4]<br />
					<span>Example: 123</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_CardForm"</h2>
				<table class="table" id="Payment_Models_Data_CardForm">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HolderName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">This parameter let you customize the holder name field on the card entry form. Per default, a mandatory holder name field is shown.</div>
	<i class="small text-muted">
Possible values: NONE, MANDATORY.<br />
					<span>Example: MANDATORY</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_CardInfo"</h2>
				<table class="table" id="Payment_Models_Data_CardInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MaskedNumber</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Masked card number</div>
	<i class="small text-muted">
					<span>Example: 912345xxxxxx1234</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ExpYear</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Year of expiration</div>
	<i class="small text-muted">
					<span>Example: 2015</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ExpMonth</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Month of expiration (eg 9 for September)</div>
	<i class="small text-muted">
					<span>Example: 9</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HolderName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the card holder (if known)</div>
	<i class="small text-muted">
					<span>Example: John Doe</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CountryCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">ISO 2-letter country code of the card origin (if available)</div>
	<i class="small text-muted">
					<span>Example: CH</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>HashValue</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The HashValue, if the hash generation is configured for the customer.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_ClientInfo"</h2>
				<table class="table" id="Payment_Models_Data_ClientInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ShopInfo</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name and version of the shop software</div>
	<i class="small text-muted">
Iso885915[1..100]<br />
					<span>Example: My Shop</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OsInfo</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the operating system</div>
	<i class="small text-muted">
Iso885915[1..100]<br />
					<span>Example: Windows Server 2013</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_DccInfo"</h2>
				<table class="table" id="Payment_Models_Data_DccInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PayerAmount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount in payer’s currency</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_DirectDebitInfo"</h2>
				<table class="table" id="Payment_Models_Data_DirectDebitInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MandateId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The unique Mandate reference, required for german direct debit payments.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CreditorId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Creditor id, required for german direct debit payments.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_InstallmentOptions"</h2>
				<table class="table" id="Payment_Models_Data_InstallmentOptions">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Initial</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">If set to true, the authorization may later be referenced for installment followup transactions.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_InvoiceInfo"</h2>
				<table class="table" id="Payment_Models_Data_InvoiceInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Payee</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_BankAccount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information about the payee, eg billpay, who is responsible for collecting the bill</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ReasonForTransfer</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The reason for transfer to be stated when paying the invoice (transfer of funds)</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DueDate</strong><br />
	<span class="text-muted small">
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The date by which the invoice needs to be settled</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Notification"</h2>
				<table class="table" id="Payment_Models_Data_Notification">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MerchantEmail</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Email address to which a confirmation email will be sent for successful authorizations.</div>
	<i class="small text-muted">
					<span>Example: merchant@saferpay.com</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PayerEmail</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Email address to which a confirmation email will be sent for successful authorizations.</div>
	<i class="small text-muted">
					<span>Example: merchant@saferpay.com</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>NotifyUrl</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Url to which Saferpay will send the asynchronous confirmation for the transaction. Supported schemes are http and https. You also have to make sure to support the GET-method.</div>
	<i class="small text-muted">
					<span>Example: https://merchanthost/notify</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PartialCapture"</h2>
				<table class="table" id="Payment_Models_Data_PartialCapture">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Type</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">'PARTIAL' if more captures should be possible later on, 'FINAL' if no more captures will be done on this authorization.</div>
	<i class="small text-muted">
Possible values: PARTIAL, FINAL.<br />
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderPartId</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Must be unique. It identifies each individual step and is especially important for follow-up actions such as refund.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: kh9ngajrfe6wfu3d0c</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Payer"</h2>
				<table class="table" id="Payment_Models_Data_Payer">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IpAddress</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">IPv4 address of the card holder / payer if available. Dotted quad notation.</div>
	<i class="small text-muted">
					<span>Example: 111.111.111.111</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>LanguageCode</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Language to be used if Saferpay displays something to the payer.

Code-List:
 de - German
 en - English
 fr - French
 da - Danish
 cs - Czech
 es - Spanish
 hr - Croatian
 it - Italian
 hu - Hungarian
 nl - Dutch
 no - Norwegian
 pl - Polish
 pt - Portuguese
 ru - Russian
 ro - Romanian
 sk - Slovak
sl - Slovenian
 fi - Finnish
sv - Swedish
 tr - Turkish
 el - Greek
 ja - Japanese
 zh - Chinese</div>
	<i class="small text-muted">
					<span>Example: de</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DeliveryAddress</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Address">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the payers delivery address</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BillingAddress</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Address">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Information on the payers billing address</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PayerInfo"</h2>
				<table class="table" id="Payment_Models_Data_PayerInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IpAddress</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">IPv4 address of the card holder / payer if available. Dotted quad notation.</div>
	<i class="small text-muted">
					<span>Example: 111.111.111.111</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IpLocation</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The location the IpAddress, if available. This might be a valid country code or a special value for 'non-country' locations (anonymous proxy, satellite phone, ...).</div>
	<i class="small text-muted">
					<span>Example: NZ</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DeliveryAddress</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Address">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px"></div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BillingAddress</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Address">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px"></div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Payment"</h2>
				<table class="table" id="Payment_Models_Data_Payment">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount data (currency, value, etc.)</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unambiguous order identifier defined by the merchant/ shop. This identifier might be used as reference later on.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Description</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">A human readable description provided by the merchant that can be displayed in web sites.</div>
	<i class="small text-muted">
Utf8[1..1000]<br />
					<span>Example: Description</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PayerNote</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Text which will be printed on payere's debit note. Supported by SIX Acquiring. No guarantee, that it will show up on the payer's debit note, because his bank has to support it too.
 Please note, that maximum allowed characters are rarely supported. It's usually around 10-12.</div>
	<i class="small text-muted">
Utf8[1..50]<br />
					<span>Example: Payernote</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MandateId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Mandate reference of the payment. Needed for German direct debits (ELV) only. The value has to be unique.</div>
	<i class="small text-muted">
Id[1..35]<br />
					<span>Example: MandateId</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Options</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Specific payment options</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Recurring</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RecurringOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Recurring options – cannot be combined with Installment.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Installment</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_InstallmentOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Installment options – cannot be combined with Recurring.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PaymentMeans"</h2>
				<table class="table" id="Payment_Models_Data_PaymentMeans">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Card</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Card">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Card data</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BankAccount</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_BankAccount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Bank account data for direct debit transaction</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Alias</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_Alias">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Alias data if payment means was registered with Secure Card Data before.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PaymentMeansInfo"</h2>
				<table class="table" id="Payment_Models_Data_PaymentMeansInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Brand</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Brand">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Brand information</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DisplayText</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Means of payment formatted / masked for display purposes</div>
	<i class="small text-muted">
					<span>Example: DisplayText</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Wallet</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of Wallet, if the transaction was done by a wallet</div>
	<i class="small text-muted">
					<span>Example: MasterPass</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Card</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_CardInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Card data</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>BankAccount</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_BankAccountInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Bank account data for direct debit transactions.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PaymentOptions"</h2>
				<table class="table" id="Payment_Models_Data_PaymentOptions">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PreAuth</strong><br />
	<span class="text-muted small">
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Used to flag the transaction as a Pre-Athorization. This type of transaction is only supported with the following Acquiring: SIX Payment Services, B+S Card Service, ConCardis, Airplus and -after an agreement- with American Express.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_PaymentPagePayment"</h2>
				<table class="table" id="Payment_Models_Data_PaymentPagePayment">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount data (currency, value, etc.)</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unambiguous order identifier defined by the merchant/ shop. This identifier might be used as reference later on.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Description</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">A human readable description provided by the merchant that will be displayed in Payment Page.</div>
	<i class="small text-muted">
Utf8[1..1000]<br />
					<span>Example: Description of payment</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PayerNote</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Text which will be printed on payere's debit note. Supported by SIX Acquiring. No guarantee, that it will show up on the payer's debit note, because his bank has to support it too.
 Please note, that maximum allowed characters are rarely supported. It's usually around 10-12.</div>
	<i class="small text-muted">
Utf8[1..50]<br />
					<span>Example: Payernote</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>MandateId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Mandate reference of the payment. Needed for German direct debits (ELV) only. The value has to be unique.</div>
	<i class="small text-muted">
Id[1..35]<br />
					<span>Example: MandateId</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Options</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_PaymentOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Specific payment options</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Recurring</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RecurringOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Recurring options – cannot be combined with Installment.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Installment</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_InstallmentOptions">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Installment options – cannot be combined with Recurring.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_RecurringOptions"</h2>
				<table class="table" id="Payment_Models_Data_RecurringOptions">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Initial</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">If set to true, the authorization may later be referenced for recurring followup transactions.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Redirect"</h2>
				<table class="table" id="Payment_Models_Data_Redirect">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>RedirectUrl</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Redirect-URL. Used to either redirect the payer or let him enter his means of payment.</div>
	<i class="small text-muted">
					<span>Example: https://www.saferpay.com/VT2/api/...</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PaymentMeansRequired</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">True, if the given URL must be used as the target of a form containing card data entered by the card holder. If ‘false’, either a GET or POST redirect without additional data may be performed.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Refund"</h2>
				<table class="table" id="Payment_Models_Data_Refund">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount data</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Reference defined by the merchant.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Description</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Description provided by merchant</div>
	<i class="small text-muted">
Utf8[1..1000]<br />
					<span>Example: Description</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_RegisterAlias"</h2>
				<table class="table" id="Payment_Models_Data_RegisterAlias">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>IdGenerator</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id generator to be used by Saferpay. 'circle' only if configured for this merchant.</div>
	<i class="small text-muted">
Possible values: MANUAL, RANDOM, RANDOM_UNIQUE.<br />
					<span>Example: manual</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Id</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Alias id to be used for registration if not generated by Saferpay. Mandatory if IdGenerator is 'manual'</div>
	<i class="small text-muted">
Id[1..40]<br />
					<span>Example: alias35nfd9mkzfw0x57iwx</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Lifetime</strong><br />
	<span class="text-muted small">
		integer
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Number of days this card is to be stored within Saferpay. If not filled, the standard lifetime will be used.</div>
	<i class="small text-muted">
Range: inclusive between 1 and 1600<br />
					<span>Example: 1000</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_RegistrationErrorInfo"</h2>
				<table class="table" id="Payment_Models_Data_RegistrationErrorInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ErrorName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name / id of the error.</div>
	<i class="small text-muted">
					<span>Example: ErrorName</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>ErrorMessage</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Description of the error.</div>
	<i class="small text-muted">
					<span>Example: ErrorMessage</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_RegistrationResult"</h2>
				<table class="table" id="Payment_Models_Data_RegistrationResult">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Success</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Result of registration</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Alias</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_AliasInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">If Success is 'true', information about the alias</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Error</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_RegistrationErrorInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">If Success is 'false', information on why the registration failed</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_ReturnUrls"</h2>
				<table class="table" id="Payment_Models_Data_ReturnUrls">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Success</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Return url for successful transaction</div>
	<i class="small text-muted">
					<span>Example: https://merchanthost/success</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Fail</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Return url for failed transaction</div>
	<i class="small text-muted">
					<span>Example: https://merchanthost/fail</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Abort</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Return url for transaction aborted by the payer.</div>
	<i class="small text-muted">
					<span>Example: https://merchanthost/abort</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Styling"</h2>
				<table class="table" id="Payment_Models_Data_Styling">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>CssUrl</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Custom styling resource (url) which will be referenced in web pages displayed by Saferpay to apply your custom styling.
 This file must be hosted on a SSL/TLS secured web server (the url must start with https://).</div>
	<i class="small text-muted">
					<span>Example: https://merchanthost/merchant.css</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Theme</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">This parameter let you customize the appearance of the displayed payment pages. Per default a lightweight responsive styling will be applied.
 If you don't want any styling use 'NONE'.</div>
	<i class="small text-muted">
Possible values: DEFAULT, SIX, NONE.<br />
					<span>Example: DEFAULT</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_ThreeDsInfo"</h2>
				<table class="table" id="Payment_Models_Data_ThreeDsInfo">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Authenticated</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Indicates, whether the payer has successfuly authenticated him/herself or not.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>LiabilityShift</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Indicates whether liability shift to issuer is possible or not. Not present if PaymentMeans container was not present in Initialize request. True, if liability shift to issuer is possible, false if not (only SSL transaction).
Please note, that the authentification can be true, but the liabilityshift is false. The issuer has the right to deny the liabiliy shift during the authorization. You can continue to capture the payment here, but we recommend to cancel unsecure payments.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Xid</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">BASE64 encoded value, given by the MPI. References the 3D Secure process.</div>
	<i class="small text-muted">
					<span>Example: ARkvCgk5Y1t/BDFFXkUPGX9DUgs=</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>VerificationValue</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Mandatory if Authenticated is true</div>
	<i class="small text-muted">
					<span>Example: AAABBIIFmAAAAAAAAAAAAAAAAAA=</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Transaction"</h2>
				<table class="table" id="Payment_Models_Data_Transaction">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Type</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Type of transaction. One of 'PAYMENT', 'REFUND'</div>
	<i class="small text-muted">
Possible values: PAYMENT, REFUND.<br />
					<span>Example: PAYMENT</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Status</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Current status of the transaction. One of 'AUTHORIZED', 'CAPTURED'</div>
	<i class="small text-muted">
Possible values: AUTHORIZED, CAPTURED.<br />
					<span>Example: AUTHORIZED</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Id</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unique Saferpay transaction id. Used to reference the transaction in any further step.</div>
	<i class="small text-muted">
					<span>Example: K5OYS9Ad6Ex4rASU1IM1b3CEU8bb</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Date</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		date
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Date / time of the authorization</div>
	<i class="small text-muted">
					<span>Example: 2011-09-23T14:57:23.023+02.00</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Amount</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		
			<a class="type-details in" href="#Payment_Models_Data_Amount">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Amount (currency, value, etc.) that has been authorized.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">OrderId given with request</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>AcquirerName</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Name of the acquirer</div>
	<i class="small text-muted">
					<span>Example: AcquirerName</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>AcquirerReference</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Reference id of the acquirer (if available)</div>
	<i class="small text-muted">
					<span>Example: AcquirerReference</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>DirectDebit</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_DirectDebitInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Direct Debit information, if applicable</div>
	<i class="small text-muted">
					<span>Example: AcquirerReference</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Invoice</strong><br />
	<span class="text-muted small">
		
			<a class="type-details in" href="#Payment_Models_Data_InvoiceInfo">container</a>
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Invoice information, if applicable</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_TransactionReference"</h2>
				<table class="table" id="Payment_Models_Data_TransactionReference">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>TransactionId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Id of the referenced transaction.</div>
	<i class="small text-muted">
AlphaNumeric[1..64]<br />
					<span>Example: 723n4MAjMdhjSAhAKEUdA8jtl9jb</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>OrderId</strong><br />
	<span class="text-muted small">
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Unambiguous OrderId of the referenced transaction.</div>
	<i class="small text-muted">
Id[1..80]<br />
					<span>Example: c52ad18472354511ab2c33b59e796901</span>
	</i>
</td>
							</tr>
					</tbody>
				</table>
			<h2>Container "Payment_Models_Data_Wallet"</h2>
				<table class="table" id="Payment_Models_Data_Wallet">
					<tbody>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>Type</strong><br />
	<span class="text-muted small">
			<span>
				<span class="text-mandatory">mandatory</span>,
			</span>
		string
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">The type of the wallet.</div>
	<i class="small text-muted">
Possible values: MASTERPASS.<br />
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>PaymentMethods</strong><br />
	<span class="text-muted small">
		string[]
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">May be used to restrict the brands which should be allowed. If not sent, we use all brands configured on this terminal.</div>
	<i class="small text-muted">
Possible values: AMEX, BANCONTACT, BONUS, DINERS, DIRECTDEBIT, EPRZELEWY, EPS, GIROPAY, IDEAL, INVOICE, JCB, MAESTRO, MASTERCARD, MYONE, PAYPAL, POSTCARD, POSTFINANCE, SAFERPAYTEST, SOFORT, VISA, VPAY.<br />
					<span>Example: [&quot;VISA&quot;, &quot;MASTERCARD&quot;]</span>
	</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>RequestDeliveryAddress</strong><br />
	<span class="text-muted small">
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">Have the payer select a delivery address in his wallet. If not sent, no address is obtained from wallet.</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
							<tr>
								<td class="col-sm-4 text-right">
	<strong>EnableAmountAdjustment</strong><br />
	<span class="text-muted small">
		boolean
	</span>
</td>
<td class="col-sm-8">
	
	<div style="padding-bottom: 10px">This option is very specific for the MasterPass business scenario where the amount may be adjusted after the redirect to MasterPass and QueryPaymentMeans to allow for changes in shipping costs.
If this is set to true, DCC will not be done right away (but may be done later with an additional redirect).
DON’T USE THIS IF YOU’RE NOT SURE – IT’S PROBABLY NOT WHAT YOU WANT!</div>
	<i class="small text-muted">
			</i>
</td>
							</tr>
					</tbody>
				</table>

</div>