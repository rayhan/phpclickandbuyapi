

# Introduction #

Payment gateway api class for clickandbuy. Process transaction using Clickandbuy marchant account.

This document will explain in details how you can use this api to process transaction with a clickandbuy marchant account.

Requires: PHP5


# Details #

## Get Instance of ClickandbuyAPI class and set class options ##

```
<?php
// Load class library
include_once("clickandbuyapi.php");


// Example Configurations:
$configs = array(   'gateway' => array(
                           'gateway_url'   => 'https://eu.clickandbuy.com/newauth/http://premium-[your keys].eu.clickandbuy.com/',
                           'return_url'	=> 'http://www.yourwebsite.com/clickandbuy_return.php',
                           'sellerID' 		=> 'your sellerId',
                           'tmPassword' 	=> 'your tmPassword',
                           'dynkey'        => '',
                       ),
               'customParams' => array('myUserId', 'myItemId'),   // these parameters will be preserved.
           );

// get instance
$api = ClickandbuyAPI::getInstance($configs);

?>
```


---

### Configuration Options: list of all configuration keys ###

> - gateway
> > - array gateway information
> > - mandatory keys
> > > - gateway\_url
> > > > : string clickand buy gateway url

> > > - return\_url
> > > > : string your landing url

> > > - sellerID
> > > > : string your clickandbuy seller ID

> > > - tmPassword
> > > > : string your clickandbuy tmPassword

> > > - dynkey
> > > > : string [optional](optional.md) default ''


> - customParams
> > - array [optional](optional.md) default empty array
> > - your custom parameters to be available on landing page
> > - these data will be available on isCommited() return array.


> - autoExternalBDRID
> > - boolean [optional](optional.md) default true
> > - set false to generate externalBDRID manually

---


## Generate Clickandbuy Link ##
```
<?php

// Get your ClickandBuy transaction parameters from database 
$data = array( 
               'cb_currency' => 'EUR', 
               'lang' => 'en', 
               'cb_billing_Nation' => 'UK', 
               'price' => 100,   // price in cent
               'cb_content_name_utf' => 'Yurssite.com', 
               'cb_content_info_utf' => 'Yur product title',
               'myUserId' => 10, // optional
               'MyItemId' => 32, // optional
            );

// Construct clickandbuy payment link
$url = $api->link($data);

// now you can use this link in html anchor like:
?>

<a href="<?php echo $url; ?>">Pay using ClickandBuy</a>
```

---



## Check Your Transaction ##
Verify transaction information and store transaction data into your database with status  uncommitted.

File: transaction\_check.php
```
<?php
  // verify transaction information and return all necessary data in array
  $checkData = $api->check(false);

  if (!empty($checkData)) {
      // check and insert server transaction data into database.
          /**
           * ... database insert code here....
           */

      // redirect to landing page for second handshake
      header('Location: ' . $checkData['url']);
      exit;      
  }
?>
```
_For details dump $checkData variable and see what data are available. May be helpful in creating database schema and query_

---


## Transaction Return Page ##
This page will call soap call isExternalBDRIDCommitted as second handshake to confirm the transaction.

File: transaction\_return.php
```
<?php
// call soap method isExternalBDRIDCommitted and return all necessary information as array
$committed = $api->isCommitted();

    if (!empty($committed)) {
        // soap status
        if ($committed['return'] == ture) {
                /**
                 * ... update transaction as completed ....
                 */

            echo "Transaction successfull!";
        } else {
                /**
                 * ... update transaction as fault ....
                 */

            echo "Transaction cancelled";
        }

        $api->pr($committed); // dump result data

    } else {
        // other errors
        echo "Transaction cancelled";

        $api->pr($api->getErrors()); // dump errors for debug
    }
?>
```

---



## Tips ##

You can merge Transaction check and Transaction Return URL in same location. See the example below:
```
<?php
if (empty($_GET['result'])) {
    // ... code for transaction check page ...
} else {
    // ... code for transaction landing page ....
}
?>
```