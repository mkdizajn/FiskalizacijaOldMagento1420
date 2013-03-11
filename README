# Fiskalizacija module za stariji magento 1420

- - -

## Za šta je to?

- dodatak na Inchoo modul za fiskalizaciju za stariji magento

## Kako koristiti

- instalirati inchoo fiskalizacija modul standardno
- kopirati u folder u app -> code -> local folder ( patch old magento with new methods )
- pokrenuti manualni sql import jer negdje setup tablica puca tj. ne pokrene se zbog nekog razloga..:

```sql
--
-- Table structure for table `inchoo_fiskalizacija_cert`
--

CREATE TABLE IF NOT EXISTS `inchoo_fiskalizacija_cert` (
  `cert_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'Id',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Created At',
  `website_id` int(11) NOT NULL COMMENT 'Website Id',
  `pfx_cert` text NOT NULL COMMENT 'Original Certificate in PFX format',
  `pem_private_key` text NOT NULL COMMENT 'Private Key in PEM format',
  `pem_private_key_passphrase` text NOT NULL COMMENT 'Passphrase for Private Key in PEM format',
  `pem_public_cert` text NOT NULL COMMENT 'Public Certificate in PEM format',
  `pem_public_cert_name` text NOT NULL COMMENT 'Name of the Public Certificate in PEM format',
  `pem_public_cert_serial_number` int(11) NOT NULL COMMENT 'Serial Number of the Public Certificate in PEM format',
  `pem_public_cert_hash` text NOT NULL COMMENT 'Hash of the Public Certificate in PEM format',
  `pem_public_cert_info` text NOT NULL COMMENT 'Info from Public Certificate in PEM format',
  `valid_from` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT 'Valid From',
  `valid_to` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT 'Valid To',
  PRIMARY KEY (`cert_id`),
  UNIQUE KEY `UNQ_INCHOO_FISKALIZACIJA_CERT_WEBSITE_ID` (`website_id`),
  UNIQUE KEY `UNQ_INCHOO_FISKALIZACIJA_CERT_PEM_PUBLIC_CERT_SERIAL_NUMBER` (`pem_public_cert_serial_number`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 COMMENT='Fiscalization Certificates' AUTO_INCREMENT=3 ;

-- --------------------------------------------------------

--
-- Table structure for table `inchoo_fiskalizacija_invoice`
--

CREATE TABLE IF NOT EXISTS `inchoo_fiskalizacija_invoice` (
  `entity_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'Id',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Created At',
  `invoice_entity_id` int(11) NOT NULL COMMENT 'Invoice Id, points to sales_flat_invoice.entity_id',
  `xml_request_raw_body` text NOT NULL COMMENT 'Final XML request, non-signed ready to be signed',
  `signed_xml_request_raw_body` text NOT NULL COMMENT 'Final signed XML request, ready to be sent to fiscalization SOAP service',
  `total_request_attempts` int(11) NOT NULL COMMENT 'Number of attempts sending the request',
  `br_rac` text COMMENT 'Broj racuna',
  `oib` varchar(11) DEFAULT NULL COMMENT 'OIB firme',
  `blagajnik` text COMMENT 'Blagajnik',
  `zast_kod` varchar(32) DEFAULT NULL COMMENT 'Zastitni kod',
  `jir` varchar(36) DEFAULT NULL COMMENT 'Time when requested passed and JIR was obtained',
  `jir_obtained_at` timestamp NULL DEFAULT NULL COMMENT 'Time when requested passed and JIR was obtained',
  `last_service_response_body` text COMMENT 'Last service response body',
  `last_service_response_status` int(11) DEFAULT NULL COMMENT 'Last service response status',
  `modified_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT 'Time when entry was last modified',
  PRIMARY KEY (`entity_id`),
  UNIQUE KEY `UNQ_INCHOO_FISKALIZACIJA_INVOICE_INVOICE_ENTITY_ID` (`invoice_entity_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 COMMENT='Fiscalization Invoice' AUTO_INCREMENT=16 ;
```

- kreirati file kao template blok u /app/design/frontend/base/default/template/sales/order/print/invoice.phtml sa ovim nutra

```php
<?php foreach ($_invoices as $_invoice): 
    $oid = $_invoice->getId();
    $fis = Mage::getModel("inchoo_fiskalizacija/invoice")->getCollection();
    foreach($fis as $item){
        if ( $item["invoice_entity_id"] == $oid ){
            $jir = $item["jir"];
            $zkod = $item["zast_kod"];
            $blag = $item["blagajnik"];
            $vrijeme = $item["created_at"];
            $oib = $item["oib"];
            $aaa = $item["br_rac"];
        }
    }
?>
    <h2 class="h2"><?php echo $this->__('Invoice #%s', $_invoice->getIncrementId()) ?></h2>
        <?php /* START Custom added */ ?>
            <div>JIR: <?php echo $jir; ?></div>
            <div>Zaštitni kod: <?php echo $zkod; ?></div>
            <div>Vrijeme izdavanja računa: <?php echo $vrijeme; ?></div>
            <div>Broj računa: <?php echo $aaa; ?></div>
            <div>OIB firme: <?php echo $oib; ?></div>
            <div>Blagajnik (oznaka blagajnika OIB/naziv): <?php echo $blag; ?></div>
        <?php /* END Custom added */ ?>
```

- dodati u transaction email template liniju:

```php
{{block type='core/template' area='frontend' template='sales/fiskal.phtml' invoice=$invoice.increment_id}}
```

- za odlazni email /app/design/frontend/base/template/sales/fiskal.phtml dodati recimo ovo.. jer nemogu pristupiti $_invoice->getInchooFiskalizacijaJir() metodi..:

```php
<?php 

    $invoice_increment_id = (string)$this->getData('invoice'); // get the invoice increment id!

    $inv = Mage::getModel("sales/order_invoice")->getCollection()->addAttributeToFilter('increment_id', $invoice_increment_id )->getFirstItem();
    $fis = Mage::getModel("inchoo_fiskalizacija/invoice")->getCollection(); // fiskal table

    // get the invoice id! == $inv->getIncrementId();

    // match the fiskal with invoice id!
    foreach($fis as $item){
        if ( $item["invoice_entity_id"] == $inv->getId() ){
            $fis1 = $item["jir"]; // jir
            $fis2 = $item["zast_kod"]; // kod
            $fis3 = $item["br_rac"]; // fiskalni racun
            $fis4 = $item["blagajnik"]; // blagajnik
            $fis5 = $item["created_at"]; // vrijeme
            $fis6 = $item["oib"]; // oib firme
        }
    }
?>

<?php /* START Custom added */ ?>
    <div>JIR: <?php echo $fis1; ?></div>
    <div>Zaštitni kod: <?php echo $fis2; ?></div>
    <div>Vrijeme izdavanja računa: <?php echo $fis5; ?></div>
    <div>Broj fiskalnog računa: <?php echo $fis3; ?></div>
    <div>OIB firme: <?php echo $fis6; ?></div>
    <div>Blagajnik: <?php echo $fis4; ?></div>
<?php /* END Custom added */ ?>
```

- - -

Mozda nebi trebalo ni iteraciju cijele tablice raditi.. nego recimo pomocu ovoga:

```php

$oid = $_invoice->getId();
$fis = Mage::getModel("inchoo_fiskalizacija/invoice")->getCollection()->addFieldToFilter('invoice_entity_id', $oid )->getFirstItem();

$fis1 = $fis["jir"];
echo $fis1;
```


that's it!


k