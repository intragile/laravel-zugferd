# Laravel ZUGFeRD / Factur-X

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-zugferd/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-zugferd/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-zugferd.svg)](https://packagist.org/packages/intragile/laravel-zugferd)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-zugferd.svg)](https://packagist.org/packages/intragile/laravel-zugferd)
[![License](https://img.shields.io/packagist/l/intragile/laravel-zugferd.svg)](LICENSE)

ZUGFeRD 2.x és Factur-X 1.0 Laravelhez: a számla XML beágyazása a PDF/A-3 dokumentumba és minden profil validálása, tiszta PHP-ben.

Ez egy vékony metacsomag: nincs saját kódja, csak a [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) csomagot húzza be a használati esethez szükséges szabálycsomaggal együtt, hogy könnyen megtaláld. A teljes funkcionalitás ott van dokumentálva.

[English documentation → README.md](README.md)

## Követelmények

- PHP 8.3+ a `dom`, `libxml` és `mbstring` kiterjesztéssel (és `zlib` a PDF-hez)
- Laravel 12 vagy 13

## Telepítés

```bash
composer require intragile/laravel-zugferd
```

Az íráshoz, olvasáshoz és validáláshoz nincs mit beállítani: a Factur-X szabályok a telepített szabálycsomagban vannak. A PDF/A-3 dokumentumot te adod (a PDF megjelenítése nem része a könyvtárnak). A doctor az alapértelmezett validátort is ellenőrzi, amely betölti a Peppol szabályokat, ezért telepítsd őket egyszer. Transportot csak akkor állíts be, ha Peppolon is küldesz (lásd a teljes dokumentációt).

```bash
php artisan einvoicing:install-rules
php artisan migrate   # létrehozza a két táblát
```

Ellenőrzés:

```bash
php artisan einvoicing:doctor
```

Használat:

```php
use IntrAgile\EInvoicing\Laravel\Facades\EInvoicing;
use IntrAgile\EInvoicing\Syntax\FacturX\{FacturXPdfWriter, FacturXProfile, FacturXXmlWriter};
use IntrAgile\EInvoicing\Syntax\Ubl\UblReader;

$invoice = (new UblReader)->read(file_get_contents('invoice.xml'))->invoice;   // or one you built

$xml = (new FacturXXmlWriter)->write($invoice, FacturXProfile::En16931)->xml;
$report = EInvoicing::facturx()->validate($xml);
if (! $report->isValid()) {
    throw new RuntimeException('The Factur-X XML is invalid.');
}

$pdf = (new FacturXPdfWriter)->embed($pdfA3Bytes, $xml)->pdf;   // $pdfA3Bytes: the PDF/A-3 you rendered
file_put_contents('invoice.pdf', $pdf);
```

A könyvtár nem konvertál PDF/A-ra: PDF/A-3 bemenetet kell adni. A profilok: MINIMUM, BASIC WL, BASIC, EN 16931, EXTENDED és XRECHNUNG; a formátum a hivatalos fájlokkal és eszközökkel offline ellenőrzött, éles forgalomban nem.

## Amit behúz

`intragile/laravel-einvoicing` (a Laravel-integráció, MIT), amely hozza a `intragile/php-einvoicing` motort (MIT) és az EN 16931 szabályokat, `intragile/einvoicing-rules-en16931` (EUPL-1.2), továbbá az `intragile/einvoicing-rules-facturx` csomagot (a Factur-X sémák és szabályok, Apache-2.0).

A metacsomag maga MIT licencű és nem szállít kódot; minden behúzott csomag a saját licencét tartja meg.

## Teljes dokumentáció

Teljes dokumentáció: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#readme)

## Licenc

MIT. Lásd: [LICENSE](LICENSE).

---

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)
