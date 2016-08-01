# M-PESA API Package
[![Build Status](https://travis-ci.org/SmoDav/mpesa.svg?branch=master)](https://travis-ci.org/SmoDav/mpesa)
[![Total Downloads](https://poser.pugx.org/smodav/mpesa/d/total.svg)](https://packagist.org/packages/smodav/mpesa)
[![Latest Stable Version](https://poser.pugx.org/smodav/mpesa/v/stable.svg)](https://packagist.org/packages/smodav/mpesa)
[![Latest Unstable Version](https://poser.pugx.org/smodav/mpesa/v/unstable.svg)](https://packagist.org/packages/smodav/mpesa)
[![License](https://poser.pugx.org/smodav/mpesa/license.svg)](https://packagist.org/packages/smodav/mpesa)

This is a PHP package for the Safaricom's M-Pesa API. 
The API allows a merchant to initiate C2B (paybill via web) transactions.
The merchant submits authentication details, transaction details, callback url and callback method. 

After request submission, the merchant receives instant feedback with validity status of their requests. 
The C2B API handles customer validation and authentication via USSD push. 
The customer then confirms the transaction. If the validation of the customer fails or the customer declines the transaction, the API makes a callback to merchant. Otherwise the transaction is processed and its status is made through a callback. 

## Installation

Pull in the package through Composer.

Run `composer require smodav/mpesa`

### Laravel

When using Laravel 5, include the service provider and its alias within your `config/app.php`.

```php
'providers' => [
    SmoDav\Mpesa\MpesaServiceProvider::class,
],

'aliases' => [
    'Mpesa' => SmoDav\Mpesa\Mpesa::class,
],
```

Publish the package specific config using:
```bash
php artisan vendor:publish
```

This will publish the M-Pesa configuration file into the `config` directory as
`mpesa.php`. This file contains all the configurations required to use the package.

### Other Frameworks

To implement this package, a configuration repository is needed, thus any other
framework will need to create its own implementation of the `ConfigurationStore`
interface.

### Native PHP

When using Native PHP, you will need to modify the configuration file found under
the package's config directory. 

## Configuration

The `mpesa.php` file should be modified to meet your needs. The following are the
settings:

- demo: boolean

Possible values: true | false

Default: true

This is a flag to set the API in demo mode and use the demo timestamp
and password. When demo is set to true, set the paybill number to 898998.

- endpoint: string

Default: "https://safaricom.co.ke/mpesa_online/lnmo_checkout_server.php?wsdl"

This is the default Safaricom API endpoint to be queried for transactional
requests.

- callback_url: string

This is a fully qualified http endpoint that will be be queried by Safaricom's
API on completion or failure of the transaction. It should point to the notification
endpoint for your application where you can finalize the checkout process.

- callback_method: HTTP Request Method

Possible values: GET | POST

Default: POST

This is the request method to be used when communicating with the Callback endpoint

- paybill_number: int

Default: 898998 (Safaricom Demo Paybill)

This is a registered Paybill Number that will be used as the Merchant ID
on every transaction. This is also the account to be debited on every successful
transaction.

- passkey: string

This is the secret SAG Passkey generated by Safaricom on registration
of the Merchant's Paybill Number.

- transaction_id_handler: class

Default: '\SmoDav\Mpesa\Generator'

Provide a fully qualified class name of the Class that will be
used to generate the transaction number. This class must implement the
Transactable Interface. On every transaction, this class will be queried
to provide a transaction number to be used. The default works fine.

## Usage

### Laravel
The package comes with a helper function `mpesa()` and its respective facade `Mpesa`.
To initiate a transaction using the facade:

```php
public function checkout()
{
    $repsonse = Mpesa::request(100)->from(254722000000)->usingReferenceId(115445)->transact();
}

```

When using the ` mpesa() ` helper function, you can directly initialize the Cashier or chain
the methods just as you would when using the facade.

```php
public function checkout()
{
    $repsonse = mpesa()->request(100)->from(254722000000)->usingReferenceId(115445)->transact();
}

public function checkout()
{
    // only initialize the amount
    
    $repsonse = mpesa(100)->from(254722000000)->usingReferenceId(115445)->transact();
}

public function checkout()
{
    // initialize the amount to be deducted and the number
    
    $repsonse = mpesa(100, 254722000000)->usingReferenceId(115445)->transact();
}

public function checkout()
{
    // initialize the amount to be deducted, the number and reference id
    
    $repsonse = mpesa(100, 254722000000, 115445)->transact();
}
```

### Native PHP

When using Native PHP, all you need to do is use the Native Implementation
of the Mpesa Class:

```php
use SmoDav\Mpesa\Native\Mpesa;

public function checkout()
{
    $mpesa = new Mpesa;
    
    $repsonse = $mpesa->request(100)->from(254722000000)->usingReferenceId(115445)->transact();
}

```

## Result

The result of any methods above will be an instance of `GuzzleHttp\Psr7\Response`.

##NOTE

The use of Safaricom's demo paybill number will actually deduct the amount from your M-Pesa account. For the testing purposes please use the minimum transaction amount which is KES 10.

##License

The M-Pesa Package is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
