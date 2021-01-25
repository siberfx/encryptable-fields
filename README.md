# Eloquent model encrypted fields for Laravel

[![Latest Version on Packagist](https://img.shields.io/packagist/v/webqamdev/encryptable-fields.svg?style=flat-square)](https://packagist.org/packages/webqamdev/encryptable-fields)
[![Total Downloads](https://img.shields.io/packagist/dt/webqamdev/encryptable-fields.svg?style=flat-square)](https://packagist.org/packages/webqamdev/encryptable-fields)

Allow you to encrypt model's fields. You can add a hashed field to allow SQL query.

## Installation

You can install the package via composer:

```bash
composer require webqamdev/encryptable-fields
```

You can publish the configuration via Artisan:

```bash
php artisan vendor:publish --provider="Webqamdev\EncryptableFields\EncryptableFieldsServiceProvider"
```

## Usage

To work with this package, you need to use our `EncryptableFields` trait in your models, then override
the `$encryptable` property. This array allows you to define encryptable attributes in your model.

You can also add attributes to contain a hash of the non encrypted value, which might be useful in order to execute a
fast full match for a given searched value.

To do so, you need to use the `$encryptable` property as an associative array, where encryptable attributes are keys and
associated hashed attributes are values.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Webqamdev\EncryptableFields\Models\Traits\EncryptableFields;

class User extends Model
{
    use EncryptableFields;

    const COLUMN_LASTNAME = 'lastname';
    const COLUMN_LASTNAME_HASH = 'lastname_hash';
    const COLUMN_FIRSTNAME = 'firstname';
    const COLUMN_FIRSTNAME_HASH = 'firstname_hash';
    const COLUMN_EMAIL = 'mail';

    /**
     * The attributes that should be encrypted in database.
     * 
     * @var string[] 
     */
    protected $encryptable = [
        self::COLUMN_FIRSTNAME => self::COLUMN_FIRSTNAME_HASH,
        self::COLUMN_LASTNAME => self::COLUMN_LASTNAME_HASH,
        self::COLUMN_EMAIL,
    ];
```

To create a new model, simply do it as before:

```php
User::create(
    [
        User::COLUMN_FIRSTNAME => 'watson',
        User::COLUMN_LASTNAME => 'jack',
    ]
);
```

To find a model from a hashed value:

```php
User::where(User::COLUMN_FIRSTNAME_HASH, User::hashValue('watson'))->first();
```

or use the model's local scope:

```php
User::whereEncrypted(User::COLUMN_FIRSTNAME, 'watson')->first();
```

### Searchable encrypted values

[MySQL](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_aes-decrypt)
and [MariaDB](https://mariadb.com/kb/en/aes_decrypt/) both provide an `aes_decrypt` function, allowing to decrypt values
directly when querying. It then becomes possible to use this function to filter or sort encrypted values.

However, Laravel default encrypter only handles `AES-128-CBC` and `AES-256-CBC` cipher methods, where `MySQL`
and `MariaDB` requires `AES-128-ECB`. We're going to use two different keys.

To do so, add the following variable to your `.env` file:

```
APP_DB_ENCRYPTION_KEY=
```

and run `php artisan encryptable-fields:key-generate` command to generate a database encryption key.

⚠️ You shouldn't generate this key on your own because ciphers differ between Laravel and MySQL/MariaDB.

Then, it is required to override Laravel's default encrypter, which is done
in [DatabaseEncrypter.php](./src/Encryption/DatabaseEncrypter.php).

Include [DatabaseEncryptionServiceProvider](./src/Providers/DatabaseEncryptionServiceProvider.php) in
your `config/app.php`, so that a singleton instance will be registered in your project, under `databaseEncrypter` key:

```php
return [
    // ...

    'providers' => [
        // ...

        /*
         * Package Service Providers...
         */
        Webqamdev\EncryptableFields\Providers\DatabaseEncryptionServiceProvider::class,

        // ...
    ],
    
    // ...
];
```

Finally, override the package configuration in `encryptable-fields.php` file:

```php
return [
    // ...

    // Need to implement EncryptionInterface
    'encryption' => Webqamdev\EncryptableFields\Services\DatabaseEncryption::class,

    // ...
];
```

If you're using [Laravel Backpack](https://backpackforlaravel.com) in your project, a
trait [EncryptedSearchTrait](./src/Http/Controllers/Admin/Traits/EncryptedSearchTrait.php) provides methods to customize
search and order logics.

### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

- [Thomas Combe](https://github.com/thomascombe)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

## Laravel Package Boilerplate

This package was generated using the [Laravel Package Boilerplate](https://laravelpackageboilerplate.com).
