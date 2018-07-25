# Teknis Integrasi system OmniPay

Berikut ini adalah cara untuk mengintegrasikan system pembayaran dalam website Anda

Pengetahuan dasar:

- Basis URL:
  - Production Environment: **https://secure.omnipay.co.id/OmniPay**  
- _Merchant ID_: merupakan id yang diberikan oleh OmniPay saat anda meregistrasikan akun merchant anda kepada kami. Merchant ID ini disediakan oleh **OmniPay**
- _Order ID_: merupakan id yang anda gunakan untuk mengidentifikasikan pesanan/order dari web
    anda.
    Order ID ini disediakan oleh **System Merchant**
- _Verify Key_: merupakan unique key untuk keamanan komunikasi antara server merchant dan
    server
    OmniPay. Verify Key ini disediakan oleh **OmniPay**
- _Callback URL_: merupakan url yang menerima notifikasi bahwa sebuah Order ID sudah dibayar
    atau
    belum. Perlu diketahui bahwa server OmniPay juga akan mengirimkan Email dan notifikasi Telegram
    kepada merchant bila pembayaran sebuah Order ID sudah dilaksanakan

## Terima pembayaran Virtual Account

**Request url: _POST_ /api-v2/va/index.php**

Virtual Account adalah nomor akun yang diberikan oleh bank untuk dapat dilakukan proses transfer, baik transfer melalui ATM, Mobile Banking, SMS Banking, maupun Internet Banking.
bila transfer sudah dilaksanakan maka Server OmniPay akan memanggil Callback?Return URL web merchant bahwa pembayaran sudah dilakukan terhadap Order ID bersangkutan tersebut

Langkah sederhananya adalah sebagai berikut:
1. Webserver Merchant request nomor VA (Virtual Account) ke server OmniPay
2. Server OmniPay akan memberikan nomor VA 
3. Webserver Merchant menampilkan nomor VA ini dan batas waktu pembayarannya
4. Buyer di website merchant dapat melakukan pembayaran melalui ATM, Mobile Banking, maupun Internet Banking dengan memilih transfer ke nomor rekening / VA tersebut seperti biasa

| Field | Tipe Data | Required | Default |
|---|:---:|:---:|---|
| returnurl | string | tidak | - |
| merchantid | string | ya | - |
| orderid | string | ya | - |
| amount | integer | ya | - |
| bill_name | string | ya | - |
| bill_email | string | ya | - |
| bill_mobile | string | ya | - |
| bill_desc | string | ya | - |
| vcode | string | ya | - |
| provider | string | tidak | permata |
| expiry_minute | integer | tidak | 240 |

```php
<?php
$url = 'https://secure.omnipay.co.id/OmniPay';
$verify_key = '473597fa188235c13f7a336c3e365517';

$request = new stdClass();
$request->returnurl = 'https://my-return-url.com/return/omnipay';
$request->merchantid = 'marketingmolpay';
$request->orderid = '123';
// amount ini adalah jumlah yang ditagihkan kepada buyer 
// setelah ditambahkan dengan fee virtual account
// misalkan jumlah tagihan order adalah 10.000, maka amount = 10.000 + fee
$request->amount = 15000;
$request->bill_name = 'test';
$request->bill_email = 'test@omnipay.co.id';
$request->bill_mobile = '08986512345';
$request->bill_desc = 'testing va payment';
$request->expiry_minute = 240;

$request->vcode = md5($request->amount . $request->merchantid . 
    $request->orderid . $verify_key);

// asumsi fungsi post sudah didefinisikan sebelumnya
// fungsi post ini mengirimkan request POST dengan body berbentuk JSON dari $request
$response = post($url . '/api-v2/va/index.php', $request);

// proses response yang didapatkan seperti biasa, 
// response ini dalam bentuk json sesuai keterangan
```

- vcode = md5(amount + merchantid + orderid + verfiy_key), vcode merupakan nilai md5 dari konkatenasi amount, merchantid, orderid, dan verify_key
- provider merupakan pilihan bank penyedia virtual account, bisa diisi dengan permata/artajasa/bca/mandiri/cimb
- expiry_minute merupakan nilai dalam menit untuk masa berlakunya virtual account, misal jam 10.00 transaksi dilakukan dan expiry_minute diisi 60, 
maka sejak jam 11.00 virtual account tersebut sudah expired
- Semua data yang dikirimkan adalah dalam bentuk **JSON**  

**response** yang didapat adalah dalam bentuk json object dimana field-fieldnya adalah sebagai berikut

| Field | Jenis | Keterangan |
| --- | --- | --- |
| va | String | Nomor Virtual Account |
| bank | String | jenis nomor rekening (permata / artajasa / bca / mandiri dst..)
| date | String | tanggal dan waktu tercatatnya transaksi |
| amount | Integer | Jumlah yang harus dibayarkan |
| duedate | String | tanggal dan waktu dimana virtual account tidak dapat dipergunakan lagi


## Terima pembayaran Kartu Kredit

**Request URL : _GET_ /pay/{$merchantid}/**

Menerima pembayaran kartu kredit Visa dan Mastercard dilakukan pada halaman yang aman.
Oleh karena itu, developer hanya perlu memberikan GET parameter
yang nantinya akan di pergunakan dalam halaman pembayaran yang disediakan oleh **OmniPay**

Parameter-parameter yang diperlukan adalah sebagai berikut:

| Field | Jenis | Required | Keterangan |
| --- | --- | --- |
| amount | Integer | Ya | Jumlah Tagihan (Termasuk fee) |
| cur | String | Tidak | Mata uang, bila tidak diisi, default IDR |
| orderid | String | Ya | Nomor Order dari Merchant |
| bill_name | String | Ya | Nama buyer |
| bill_email | String | Ya | Email dari buyer |
| bill_mobile | String | Ya | Nomor telepon buyer |
| bill_desc | String | Ya | Keterangan tentang penjualan |
| country | String | Ya | diisi "ID" |
| returnurl | String | Tidak | Url yang akan di notifikasi oleh sistem tunjukkan saat buyer menyelesaikan / membatalkan pembayaran |
| vcode | String | Ya | md5(amount + merchantid + orderid + verfiy_key) |

```php
<?php
$merchantid = 'marketingmolpay';
$verifyKey = '473597fa188235c13f7a336c3e365517';
$url = "https://secure.omnipay.co.id/OmniPay/pay/$merchantid/?";

$amount = 15000;
$cur = 'IDR';
$orderid = 'your ourder id';
$name = 'very cool name';
$email = 'demo@omnipay.co.id';
$mobile = '08987171771';
$description = 'This is only a test';
$returnurl = 'http://my-return-url.domain.tld/return.php';
$vcode = md5($amount . $merchantid . $orderid . $verifyKey);

// since the parameter will be used in a GET request, we need to
// urlencode them
$amount = urlencode($amount);
$cur = urlencode($cur);
$orderid = urlencode($orderid);
$name = urlencode($name);
$email = urlencode($email);
$mobile = urlencode($mobile);
$description = urlencode($description);
$returnurl = urlencode($returnurl);

// show that link anywhere the customer should pay
echo "<a href=\"$url"; 
echo "amount=$amount&"; 
echo "cur=$cur&"; 
echo "orderid=$orderid&"; 
echo "bill_name=$name&"; 
echo "bill_email=$email&"; 
echo "bill_mobile=$mobile&"; 
echo "bill_desc=$description&"; 
echo "country=ID"; 
echo "returnurl=$returnurl&"; 
echo "vcode=$vcode&";  
echo "langcode=id\"> Pay Now </a>";      
```

Contoh PHP berikut ini merupakan contoh menampilkan sebuah link yang nantinya akan di _click_ oleh _buyer_ untuk
melakukan pembayaran dengan menggunakan Kartu Kredit

### Notifikasi pembayaran Kartu Kredit

#### RETURN URL

Setelah buyer click pada tombol bayar, mereka akan diarahkan pada halaman OmniPay untuk memproses pembayaran kartu kredit
buyer, setelah buyer menyelesaikan pembayaran, maka server OmniPay akan melakukan POST request kepada *returnurl* yang 
telah di definisikan oleh merchant sebelumnya

Return URL tersebut akan mendapatkan request dengan field POST sebagai berikut:

| Field | Jenis | Keterangan |
|---|---|---|
|domain|String|The merchant id|
|amount|Integer|The transaction amount in one bill|
|orderid|String|The bill/invoice number|
|appcode|String|Bank approval code| 
|tranID|String|Transaction ID for tracking purpose domain Alpha-numeric  Merchant ID|
|status|00 or 11|Status of transaction: 00 - success 11 - failure| 
|error_code|String|Error code for failure transaction (if any)| 
|error_desc|String|Error description for failure transaction (if any)| 
|currency|String|Mata uang pembayaran (selalu IDR)| 
|paydate|Date/Time YYYY-MM-DD HH:mm:ss|Date time of the transaction| 
|channel|String|Depends to the payment method| 
|skey|String|Hashed string to verify whether the transaction is from a valid source. Verify Key is required| 
|fx_amount|Number|Jumlah yang dibayarkan bila currency tdk dalam IDR (2 desimal)|
|fx_currency|String|Mata uang bila tagihan bukan dalam IDR|
|fx_rate|Number|Nilai tukar terhadap misal bila fx_currency = USD, maka nilainya adalah nilai 1USD dalam IDR (4 desimal)|
|fx_skey|String|md5(verifyKey . fx_amount . fx_currency . fx_rate . tranID . orderid)|

```php
<?php 
 
$verifyKey ="473597fa188235c13f7a336c3e365517";

$tranID = $_POST['tranID'];
$orderid = $_POST['orderid'];
$status = $_POST['status'];
$domain = $_POST['domain'];
$amount = $_POST['amount'];
$currency = $_POST['currency'];
$appcode = $_POST['appcode'];
$paydate = $_POST['paydate'];
$skey = $_POST['skey'];
$fx_amount = $_POST['fx_amount'];
$fx_currency = $_POST['fx_currency'];
$fx_rate = $_POST['fx_rate'];
$fx_key = $_POST['fx_key'];

// check apakah post ini benar dari OmniPay server
$key0 = md5( $tranID . $orderid.$status.$domain.$amount.$currency ); 
$key1 = md5( $paydate.$domain.$key0.$appcode.$verifyKey );

// asumsi bahwa pembayaran dalam IDR, jadi default pemeriksaan parameter fx adalah "true"
$fx_ok = true;

// kalau server mengirimkan $_POST['fx_key'] artinya tagihan tidak dalam IDR
if(!empty($fx_key)) {
    $calc_fx_key = md5($verifyKey . $fx_amount . $fx_currency . $fx_rate . $tranID . $orderid);    
    $fx_ok = $calc_fx_key === $fx_ok;
}

if( $skey === $key1 && $fx_ok) {
    // we are having a valid transaction return, let's check for the status..
    if($status === '00') {
        // payment is successfull, update the order accordingly
        // important!!! check the orderid, check the amount!
    } else {
        // payment is not success.. perhaps check for the POSTed error_code and error_desc 
    }
} else {
    // Invalid security key, it is possible that the data is not sent from OmniPay's server
    // !!! Beware !!!
}
 
``` 

#### CALLBACK URL

Ada kalanya dimana returnurl tidak dipergunakan, dalam hal ini misalkan Buyer langsung membayar lewat tagihan Invoice (email).
Merchant bisa memanfaatkan fasilitas CALLBACK URL untuk memantau apakah suatu invoice sudah dibayarkan atau belum.

Fasilitas callback url ini harus diaktifkan terlebih dahulu di dalam dashboard merchant pada halaman https://secure.omnipay.co.id/OmniPay

Server OmniPay melakukan callback dengan request berbentuk **POST**. Dari **POST** request tersebut diharapkan didapatkan response yang **HANYA** berisi

`CBTOKEN:MPSTATOK`

Isi callback tersebut adalah sebagai berikut:

| Field | Jenis | Keterangan |
|---|---|---|
|nbcb|Integer|Selalu berisi **1**|
|domain|String|The merchant id|
|amount|Integer|The transaction amount in one bill|
|orderid|String|The bill/invoice number|
|appcode|String|Bank approval code| 
|tranID|String|Transaction ID for tracking purpose domain Alpha-numeric  Merchant ID|
|status|00 or 11|Status of transaction: 00 - success 11 - failure| 
|error_code|String|Error code for failure transaction (if any)| 
|error_desc|String|Error description for failure transaction (if any)| 
|currency|String|Mata uang pembayaran| 
|paydate|Date/Time YYYY-MM-DD HH:mm:ss|Date time of the transaction| 
|skey|String|Hashed string to verify whether the transaction is from a valid source. Verify Key is required| 
|fx_amount|Number|Jumlah yang dibayarkan bila currency tdk dalam IDR (2 desimal)|
|fx_currency|String|Mata uang bila tagihan bukan dalam IDR|
|fx_rate|Number|Nilai tukar terhadap misal bila fx_currency = USD, maka nilainya adalah nilai 1USD dalam IDR (4 desimal)|
|fx_skey|String|md5(verifyKey . fx_amount . fx_currency . fx_rate . tranID . orderid)|

```php
<?php 
 
$verifyKey ="473597fa188235c13f7a336c3e365517";

$nbcb = $_POST['nbcb'];
$tranID = $_POST['tranID'];
$orderid = $_POST['orderid'];
$status = $_POST['status'];
$domain = $_POST['domain'];
$amount = $_POST['amount'];
$currency = $_POST['currency'];
$appcode = $_POST['appcode'];
$paydate = $_POST['paydate'];
$skey = $_POST['skey']; 

$fx_amount = $_POST['fx_amount'];
$fx_currency = $_POST['fx_currency'];
$fx_rate = $_POST['fx_rate'];
$fx_key = $_POST['fx_key'];

// check apakah post ini benar dari OmniPay server
$key0 = md5( $tranID . $orderid.$status.$domain.$amount.$currency ); 
$key1 = md5( $paydate.$domain.$key0.$appcode.$verifyKey );

// asumsi bahwa pembayaran dalam IDR, jadi default pemeriksaan parameter fx adalah "true"
$fx_ok = true;

// kalau server mengirimkan $_POST['fx_key'] artinya tagihan tidak dalam IDR
if(!empty($fx_key)) {
    $calc_fx_key = md5($verifyKey . $fx_amount . $fx_currency . $fx_rate . $tranID . $orderid);    
    $fx_ok = $calc_fx_key === $fx_ok;
}

if( $skey === $key1 && $fx_ok) {
    // response ini diperlukan oleh OmniPay server untuk mencatat bahwa server merchant sudah menerima notifikasi
    // callback sebuah pembayaran 
    echo "CBTOKEN:MPSTATOK";
    
    // we are having a valid transaction return, let's check for the status..
    if($status === '00') {
        // payment is successfull, update the order accordingly
        // important!!! check the orderid, check the amount!
    } else {
        // payment is not success.. perhaps check for the POSTed error_code and error_desc 
    }
} else {
    // Invalid security key, it is possible that the data is not sent from OmniPay's server
    // !!! Beware !!!
}

```


## Update Log

- Yohan, 07 Feb 2018 : Initial V2 system integration
- Yohan, 13 Feb 2018 : Initial doc Credit Card integration
- Yohan, 25 July 2018 : Menambahkan verifikasi untuk transaksi dengan AMOUNT bukan dalam IDR, Menambahkan parameter returnurl