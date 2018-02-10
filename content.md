# Teknis Integrasi system OmniPay

**Update Log**
- Yohan, 07 Feb 2018 : Initial V2 system integration
---------------------------------------------------------

Berikut ini adalah cara untuk mengintegrasikan system pembayaran dalam website Anda

Pengetahuan dasar:

- Basis URL:
  - Production Environment: **https://secure.omnipay.co.id/OmniPay**
  - Development Environment: **https://dev.secure.omnipay.co.id/OmniPay**
- Semua data yang dikirimkan adalah dalam bentuk **JSON**  
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

**Request url: /api-v2/va/index.php**

Virtual Account adalah nomor akun yang diberikan oleh bank untuk dapat dilakukan proses transfer, baik transfer melalui ATM, Mobile Banking, SMS Banking, maupun Internet Banking.
bila transfer sudah dilaksanakan maka Server OmniPay akan memanggil Callback URL web merchant bahwa pembayaran sudah dilakukan terhadap Order ID bersangkutan tersebut

Langkah sederhananya adalah sebagai berikut:
1. Webserver Merchant request nomor VA (Virtual Account) ke server OmniPay
2. Server OmniPay akan memberikan nomor VA 
3. Webserver Merchant menampilkan nomor VA ini dan batas waktu pembayarannya
4. Buyer di website merchant dapat melakukan pembayaran melalui ATM, Mobile Banking, maupun Internet Banking dengan memilih transfer ke nomor rekening / VA tersebut seperti biasa

| Field | Tipe Data | Required | Default |
|---|:---:|:---:|---|
| merchantid | string | y | - |
| orderid | string | y | - |
| amount | integer | y | - |
| bill_name | string | y | - |
| bill_email | string | y | - |
| bill_mobile | string | y | - |
| bill_desc | string | y | - |
| vcode | string | y | - |
| provider | string | n | permata |
| expiry_minute | integer | n | 240 |


- vcode = md5(amount + merchantid + orderid + verfiy_key), vcode merupakan nilai md5 dari konkatenasi amount, merchantid, orderid, dan verify_key
- provider merupakan pilihan bank penyedia virtual account, bisa diisi dengan permata/artajasa/bca/mandiri/cimb
- expiry_minute merupakan nilai dalam menit untuk masa berlakunya virtual account, misal jam 10.00 transaksi dilakukan dan expiry_minute diisi 60, 
maka sejak jam 11.00 virtual account tersebut sudah expired

```php
$url = 'https://secure.omnipay.co.id/OmniPay'
$verify_key = '473597fa188235c13f7a336c3e365517';

$request = new stdClass();
$request->merchantid = 'marketingmolpay';
$request->orderid = '123';
// amount ini adalah jumlah yang ditagihkan kepada buyer 
// setelah ditambahkan dengan fee virtual account
// misalkan jumlah tagihan order adalah 10.000, maka amount = 10.000 + fee
$request->amount = 15000;
$request->bill_name = 'yohan';
$request->bill_email = 'yohan@omnipay.co.id';
$request->bill_mobile = '08986525365';
$request->bill_desc = 'testing va payment';
$request->expiry_minute = 240;

$request->vcode = $request->amount . $request->merchantid . 
    $request->orderid . $verify_key;

// asumsi fungsi post sudah didefinisikan sebelumnya
// fungsi post ini mengirimkan request POST dengan body berbentuk JSON
$response = post($url . '/api-v2/va', $request);

// proses response yang didapatkan seperti biasa, 
// response ini dalam bentuk json sesuai keterangan
```

**response** yang didapat adalah dalam bentuk json object dimana field-fieldnya adalah sebagai berikut

| Field | Jenis | Keterangan |
| --- | --- | --- |
| va | String | Nomor Virtual Account |
| bank | String | jenis nomor rekening (permata / artajasa / bca / mandiri dst..)
| date | String | tanggal dan waktu tercatatnya transaksi |
| amount | Integer | Jumlah yang harus dibayarkan |
| duedate | String | tanggal dan waktu dimana virtual account tidak dapat dipergunakan lagi


## Terima pembayaran Transfer Bank

**!!!Dokumentasi masih on progress!!!**

Metode transfer **tanpa** menggunakan nomor akun Virtual, dalam hal ini pembeli akan mentransfer sejumlah uang
dengan nomor unik tertentu kepada rekening bank yang sudah ditunjuk oleh OmniPay, metode ini bisa dipilih bila dikehendaki adanya biaya yang lebih murah daripada metode Virtual Account

## Terima pembayaran Kartu Kredit

**!!!Dokumentasi masih on progress!!!**

Menerima pembayaran kartu kredit Visa dan Mastercard

## Integrasi Inline

**!!!Dokumentasi masih on progress!!!**

Integrasi Instan adalah dimana merchant menampilkan halaman pemilihan metode-metode pembayaran yang disediakan oleh OmniPay secara langsung dalam halaman website checkout

Integrasi ini dapat dilakukan secara "Inline" yakni dimunculkan dalam halaman checkout website merchant, atau merchant merujuk kepada halaman khusus dari Omnipay untuk melakukan pembayaran
disana