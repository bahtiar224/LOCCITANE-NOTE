# Fixing error mitra10

#### blibli awb not found have reset status to ready_for_ship

```
SELECT ss.channel_order_increment_id, ss.status, ssg.status as status_grid, ssg.channel_shipping_method, ss.marketplace_order_status, sst.track_number
FROM sales_shipment ss
LEFT JOIN sales_shipment_grid ssg ON ss.increment_id = ssg.increment_id
LEFT JOIN sales_shipment_track sst ON ss.entity_id = sst.parent_id
WHERE ss.channel_order_increment_id IN ('orderNo-packageId);
```

```
SELECT *
FROM oms_history_api
WHERE responds LIKE '%orderNo%';
```

#### query for city duplicate jne issue
```SELECT city, kecamatan, COUNT(*) FROM `city` GROUP BY city, kecamatan HAVING COUNT(*) > 1```

explain :
pake endpoint baru (pickupcashless) mba 
@femmy
 , findingnya ini sih :
order_id jika terdapat alphabet tetapi ada character yang lower case sama uppercase misal Mitra10-000029365 status success tetapi cnote_no : null
jika value dalam suatu attribute melebihi size di jne, muncul error Content not confirm our security Policy.
(edited)

#### kalo order blibli pake shipping method dikriim oleh seller awb diisi lewat oms
selain itu kosongin aja, soal e dapet dari blibli (misal JNE)

#### mas Fahmi, terkait awb yang udah ada di seller center blibli tapi di oms belum
bisa di pull order aja, bisa lewat shipment dashboard terus masuk ke order queue > fetch order > pilih tanggal per kapan order itu dibuat
login nya pake user@swiftoms.com/swiftoms
dan kalo udah kejadian berulang kali bisa lapor ke zeus aja

#### Step check test case idexpress using packing kayu
- mastiin idexpress type (shipping_method sama custom_order_attributes)
- is_packing_kayu = 1
- proses order biasa allocation -> shipment
- check shipment method ada packing kayu
- proses order sampe ke book courier
- mitraoms:idexpress:booking
- check data in oms_idexpress_queue make sure awb_no sama sorting_code
- make sure data from oms_history_api -> insured, express_type, item_value
- item_value (harga barang * qty)

#### Solution issue export order an error has occured
- yups, paling kalo error gitu nanti mereka minta contoh jsonnya
- bisa ambil di kolom request, table oms_history_api
- search aja request like '%<shipment_number>%' contoh request like '%000028851%'

#### flow update stock/price from nav
- mitraoms:decode-binary -> ubah status di oms_queue ke 1 sama insert data ke oms_queue_detail
- mitraoms:update-source:queue -> ubah status di oms_queue  ke 2 dan ubah status di oms_queue_detail  ke 1
- mitraoms:update-source:channel -> ubah status di oms_queue  ke 3 dan ubah status di oms_queue_detail  ke 2

#### step push product MP
- prepare csv for import image (base csv from channel -> mas Dimas or mas Andri)
- modify image_url in https://docs.google.com/spreadsheets/d/1icftvAZysEcbrYXkDSNNSAf-Y4WXCeXYjptc2FrmKlY/edit?hl=id#gid=2040014897 -> add base url
- download csv from spreadsheet
- import csv CPI using urapidflow (profile -> product extra import)
- prepare attribute -> channel code (unselect another mp before push), mncm_remote_category, mncm_brand, (create backup for sku with 2 mp) --> import with profile Import product mp example (https://docs.google.com/spreadsheets/d/1icftvAZysEcbrYXkDSNNSAf-Y4WXCeXYjptc2FrmKlY/edit?hl=id#gid=188409420)
- check a prepare attribute
```
SELECT cpe.sku, ea.attribute_code, cpet.value
FROM catalog_product_entity cpe
LEFT JOIN catalog_product_entity_text cpet ON cpe.entity_id = cpet.entity_id
LEFT JOIN eav_attribute ea ON cpet.attribute_id = ea.attribute_id
WHERE ea.attribute_code = 'channel_code'
AND cpe.sku IN ('sku');

SELECT cpe.sku, ea.attribute_code, cpet.value
FROM catalog_product_entity cpe
LEFT JOIN catalog_product_entity_text cpet ON cpe.entity_id = cpet.entity_id
LEFT JOIN eav_attribute ea ON cpet.attribute_id = ea.attribute_id
WHERE ea.attribute_code = 'mncm_remote_category'
AND cpe.sku IN ('sku');

SELECT cpe.sku, ea.attribute_code, cpet.value
FROM catalog_product_entity cpe
LEFT JOIN catalog_product_entity_text cpet ON cpe.entity_id = cpet.entity_id
LEFT JOIN eav_attribute ea ON cpet.attribute_id = ea.attribute_id
WHERE ea.attribute_code = 'mncm_brand'
AND cpe.sku IN ('sku');
```
- after all preparation done push product
- add 1 row in oms_queue : type = CNX_BULK_UPSERT_PRODUCT , data = array of product , status = pending
- check product in oms_marketplace_product_status and oms_marketplace_remtoe_variant
- check push in oms_history_api end_url `CNX_FEEDBACK_BULK_UPSERT_PRODUCT_171`

query for update special price update to mp
```
UPDATE oms_marketplace_product_status SET update_to_mp = 1 
WHERE marketplace_code = 'MNCM' AND sku IN ('sku')
```

```
SELECT os.sku, op.special_price, op.special_from_date as special_from_date, '2023-01-31 00:00:00' as special_to_date
FROM oms_sourcing os
LEFT JOIN oms_price op ON os.sku = op.sku AND os.loc_id = op.loc_id
WHERE os.loc_id = 2
AND op.channel_code = 'Mitra10_MNCM'
AND os.sku IN ('sku');
```

#### query to get channel code of product
```
SELECT cpe.sku, ea.attribute_code, cpet.value
FROM catalog_product_entity cpe
LEFT JOIN catalog_product_entity_text cpet ON cpe.entity_id = cpet.entity_id
LEFT JOIN eav_attribute ea ON cpet.attribute_id = ea.attribute_id
WHERE ea.attribute_code = 'channel_code'
AND cpe.sku IN ('sku');
```

https://rundeck.sirclo.net/menu/home

```
bin/magento swiftoms:mpadapter:push-product --type='queue'
bin/magento swiftoms:mpadapter:pull-feedback --feedback-type bulk_create_product
```
to check status push product mp -> oms_marketplace_product_status

#### query check status export to nav
```
SELECT entity_id, channel_order_increment_id, `export_flag`, `export_result_msg`, export_complete_flag, export_complete_msg
FROM sales_shipment
WHERE channel_order_increment_id IN ('channel_order_increment');
```

#### Fixing error ```Item (Icube\City\Model\City) with the same ID "ID-JB" already exists.```
- search in table city filter by city = 'cityname' AND kecamatan = 'kecamatanname'
- delete oldest data
- access this db
```
https://database.swiftstore.dmmy.me/?server=10.186.0.86&username=mage2user&db=m2_mitra10new&select=city_new&columns%5B0%5D%5Bfun%5D=&columns%5B0%5D%5Bcol%5D=&where%5B0%5D%5Bcol%5D=city&where%5B0%5D%5Bop%5D=%3D&where%5B0%5D%5Bval%5D=Ambalau&where%5B2%5D%5Bcol%5D=&where%5B2%5D%5Bop%5D=%3D&where%5B2%5D%5Bval%5D=&order%5B0%5D=&limit=50&text_length=100
```
- compare kecamatan_code and change kecamatan_code from db staging if kecamatan code is different
- check table oms_jne_queue for make sure the awb generated (filter by shipment_id)

jika order is_pickup tidak muncul di dashboard, kemungkinan channel_code null atau value is_pickup 0

# Fixing error loccitane
#### Failed Posting Sales Order :INVALID_FLD_VALUE details :The field custcol_aba_line_discount contained more than the maximum number ( 15 ) of characters allowed.
https://github.com/icube-mage/swiftoms-loccitane/blob/0e87100128b39a7ee0326202fd039a949696ee9c/app/code/LoccitaneOms/Netsuite/Helper/Xsl.php#L234
 https://github.com/icube-mage/swiftoms-loccitane/blob/0e87100128b39a7ee0326202fd039a949696ee9c/app/code/LoccitaneOms/Netsuite/Helper/Xsl.php#L255 (edited) 

#### note :
export progress 1 -> queue, 2 -> order header, 3 -> payment, 4 -> fulfillment, 5-> success
export status 1 -> process , 2-> success , 3->error

#### paraphase: coldparrot31

#### QUERY CHECK ORDER FAILED PUSH
```
SELECT oo.channel_order_increment_id, oo.inserted_at, oo.channel_code, so.entity_id as order_id, ss.entity_id as shipment_id, ss.increment_id, so.export_progress, so.export_status, so.netsuite_internal_id 
FROM oms_order oo
LEFT JOIN sales_order so ON oo.oms_order_id = so.entity_id
LEFT JOIN sales_shipment ss ON so.entity_id = ss.order_id
WHERE oo.channel_order_increment_id IN ('masukin channel_order_increment_id');
```

#### QUERY SET NULL EXPORT PROGRESS & STATUS
```UPDATE sales_order SET export_progress = NULL, export_status = NULL WHERE entity_id IN ('masukin entity id');```

export_progress 0, check di customer_entity email nya ada ga, di replace email sama mas reza (edited) 

#### kalo grand_total minus, di edit ke 0 saja
https://sirclo.slack.com/archives/CSJSZT20Z/p1667190507513349?thread_ts=1667188256.913159&cid=CSJSZT20Z

#### step :
create new location
Create Virtual Stock , assign location ke masing2 VS
add VS to channel in (OMS > Channel > select Channel) contoh ada di ss, harap di note ini multiselect
source akan kebuat ikut cron jam 20:45

```
SELECT oo.channel_order_increment_id, oo.inserted_at, oo.channel_code, so.entity_id as order_id, ss.entity_id as shipment_id, ss.increment_id, so.export_progress, so.export_status, so.netsuite_internal_id 
FROM oms_order oo
LEFT JOIN sales_order so ON oo.oms_order_id = so.entity_id
LEFT JOIN sales_shipment ss ON so.entity_id = ss.order_id
WHERE (so.export_status IN (1, 3) OR so.export_status IS NULL) AND so.netsuite_internal_id IS NULL AND YEAR(so.created_at) = 2022 AND MONTH(so.created_at) = 10;
```

#### Check xsl send to netsuite
- cek ke sales shipment untuk mendapatkan increment id
- insert increment_id ke export profile > profile > output format (bagian kanan atas)
- klik tombol test xsl
- compare ke oms_order table untuk mendapatkan value oms_order_id
- oms_order_id dipake di table sales_order (entity_id)
- misalkan ada perbedaan perlu cek ke table oms_order (custom_order_attributes)

#### fixing error format email salah push to netsuite
- email di ubah ditable sales_order dan sales_order_address

#### Fixing jika virtual stock tidak sync/qty salable berbeda antara oms dan channel
- go to omslite Catalog Inventory > Virtual Stock
- search channel name yang bermasalah > view
- assign location virtual stock 
context : channel gak dapet karena result karena location BECM-P belum di assign di VS (virtual stock) punya BECM-P

# Command
#### renmove generated file
```rm -rf generated/```

#### Sales channel setup
```
git clone https://github.com/icube-mage/swift-indesso.git .

git checkout develop

composer install --prefer-dist

php bin/magento setup:install --cleanup-database --base-url=http://local.indesso.testingnow.me/ --base-url-secure=https://local.indesso.testingnow.me/ \
--db-host=127.0.0.1 --db-name=indesso --db-user=root --db-password=root \
--admin-firstname=Swift --admin-lastname=Install --admin-email=ekky.ekky@sirclo.com \
--admin-user=ekkyicube --admin-password=Password123 --backend-frontname=backoffice \
--language=en_US --currency=USD --timezone=Asia/Jakarta --use-rewrites=1 --use-secure-admin=1

php bin/magento setup:upgrade

php bin/magento weltpixel:import:configurations --store=GLOBAL --file="weltpixel_configurations_admin.csv"

mysql -u root -p indesso < sql/theme_initial.sql

mysql -u root -p indesso < sql/ves_megamenu_init_v1.0_swiftdev4.sql

php bin/magento setup:upgrade && php bin/magento setup:di:compile && php bin/magento weltpixel:less:generate && \
php bin/magento setup:static-content:deploy -f en_US id_ID && php bin/magento weltpixel:css:generate --store=default && \
php bin/magento cache:flush && chmod -R 777 var/ pub/ generated/
```

#### Downgrade composer version
```composer self-update 1.9.2```

#### Change default php version to v7.2
```update-alternatives --set php /usr/bin/php7.2```

#### create admin backoffice
```bin/magento admin:user:create --admin-user=adminbo --admin-password=think2icube --admin-email=fahmi.widianto@sirclo.com --admin-firstname=fahmi --admin-lastname=widianto```

#### dump db
```mysqldump -u username -p dbname > filename.sql```

#### killed
```COMPOSER_MEMORY_LIMIT=-1 composer require <package-name>```

#### create order oms
```
bin/magento swiftoms:inventory:allocate-order
bin/magento swiftoms:create-order
bin/magento swiftoms:invoice:generate
bin/magento swiftoms:shipment:create-shipments
```

#### log code
```
$writer = new \Zend_Log_Writer_Stream(BP.'/var/log/order.log');
$logger = new \Zend_Log();
$logger->addWriter($writer);
$logger->info("");
```

#### Get latest ke version tags (yang ditentukan)
- `git checkout tags/3.32.2` <--- tag berapa
- jika tags tidak ditemukan, `git fetch` terlebih dahulu setelah fetch akan ada file local changes disitu lakukan backup file local changes kemudian `git stash`
- setelah selesai pastikan module custom ada di composer.json, jika tidak ada lakukan composer config dan composer require custom module
- kemudian lakukan `sh full_deploy.sh`