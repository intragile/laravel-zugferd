# Laravel ZUGFeRD / Factur-X

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-zugferd/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-zugferd/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-zugferd.svg)](https://packagist.org/packages/intragile/laravel-zugferd)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-zugferd.svg)](https://packagist.org/packages/intragile/laravel-zugferd)
[![License](https://img.shields.io/packagist/l/intragile/laravel-zugferd.svg)](LICENSE)

ZUGFeRD 2.x and Factur-X 1.0 for Laravel: embed the invoice XML into your PDF/A-3 and validate every profile, in pure PHP.

This is a thin metapackage: it has no code of its own and pulls in [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) together with the rule package this use case needs, so that you can find it. The full functionality is documented there.

[Magyar nyelvű leírás → README.hu.md](README.hu.md)

## Requirements

- PHP 8.3+ with `dom`, `libxml`, `mbstring` (and `zlib` for PDF)
- Laravel 12 or 13

## Quick start

```bash
composer require intragile/laravel-zugferd
```

Nothing needs configuring to write, read and validate: the Factur-X rules ship in the installed rule package. You supply the PDF/A-3 (rendering the PDF is not part of the library). The doctor also checks the default validator, which loads the Peppol rules, so install them once. Set a transport only if you also send over Peppol (see the full documentation).

```bash
php artisan einvoicing:install-rules
php artisan migrate   # create the two tables
```

Check the installation:

```bash
php artisan einvoicing:doctor
```

Usage:

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

The library does not convert to PDF/A: give it a PDF/A-3 input. The profiles are MINIMUM, BASIC WL, BASIC, EN 16931, EXTENDED and XRECHNUNG; the format is verified offline against the official files and tools, not in live exchanges.

## What it pulls in

`intragile/laravel-einvoicing` (the Laravel bridge, MIT), which brings the engine `intragile/php-einvoicing` (MIT) and the EN 16931 rules `intragile/einvoicing-rules-en16931` (EUPL-1.2), plus `intragile/einvoicing-rules-facturx` (the Factur-X schemas and rules, Apache-2.0).

The metapackage itself is MIT-licensed and ships no code; each pulled package keeps its own licence.

## Full documentation

Full documentation: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#readme)

## License

MIT. See [LICENSE](LICENSE).

---

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)
