# Batavi REST API Integration Guide

Version: 1.0  
Last updated: 23 April 2026

## Overview

This document describes the Batavi REST API endpoints used to create orders and retrieve order details. It is intended for external partners, ERP connectors, middleware platforms, and webshop integrations.

The API supports:

- Creating one or more Batavi orders from an external purchase order.
- Submitting separate shipping and invoice addresses.
- Ordering products by SKU or product ID.
- Returning warnings for configurable business checks such as price mismatches, stock issues, and invalid stock locations.
- Retrieving complete or filtered order details, including order-line status.

## Base URL

```text
https://{your-domain}/rest
```

Replace `{your-domain}` with the domain provided by the shop administrator.

## Authentication

The API supports HTTP Basic Authentication and OAuth2 Client Credentials. The shop administrator will confirm which method is enabled for your account and provide the required credentials.

### HTTP Basic Authentication

Send the username and password with every request.

```bash
curl -u "username:password" \
  "https://example.com/rest/order/list?id=336"
```

### OAuth2 Client Credentials

First request an access token using the client ID and client secret provided by the shop administrator.

```bash
curl -X POST "https://example.com/oauth.php" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

Example token response:

```json
{
  "access_token": "abc123...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Use the token in the `Authorization` header for API requests. Tokens expire after the number of seconds returned in `expires_in`.

```bash
curl -H "Authorization: Bearer abc123..." \
  "https://example.com/rest/order/list?id=336"
```

## Request Conventions

| Item | Requirement |
|------|-------------|
| Content type | Send JSON request bodies with `Content-Type: application/json`. |
| Character encoding | Use UTF-8. |
| Dates | Date-time values are returned in `YYYY-MM-DD HH:MM:SS` format. |
| Prices | Prices are returned in the shop's default currency, typically EUR. |
| VAT | Amounts with `Inc` include VAT. Amounts with `Exc`, `Excl`, or `ExclTax` exclude VAT. |

## Endpoint Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rest/order/create` | Create an order. |
| `GET` | `/rest/order/list` | Retrieve order details by order ID. |

---

## Create Order

Creates a Batavi order for the supplied customer and product lines.

```http
POST /rest/order/create
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_id` | integer | Yes | Customer login ID supplied by the shop administrator. |
| `reference_code` | string | No | External order reference, such as a purchase order number. |
| `comment` | string | No | Internal order note or delivery instruction. |
| `shipping` | object | Yes | Shipping address. |
| `invoice` | object | Yes | Invoice or billing address. |
| `products` | array | Yes | Product lines. At least one product is required. |

### Address Object

The `shipping` and `invoice` objects use the same fields.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `firstname` | string | Yes | First name. |
| `lastname` | string | Yes | Last name. |
| `street` | string | Yes | Street name. |
| `house_number` | string | Yes | House number. |
| `postcode` | string | Yes | Postal code. |
| `city` | string | Yes | City. |
| `country_iso2` | string | Yes | ISO 3166-1 alpha-2 country code, such as `NL`, `DE`, or `BE`. |
| `company` | string | No | Company name. |
| `telephone` | string | No | Phone number. |
| `email` | string | No | Email address. |
| `vat` | string | No | VAT number. |
| `coc` | string | No | Chamber of Commerce number. |
| `department` | string | No | Department name. |

### Product Object

Each product line must contain either `sku` or `product_id`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sku` | string | Conditional | Product SKU. Required when `product_id` is not supplied. |
| `product_id` | integer | Conditional | Batavi product ID. Required when `sku` is not supplied. |
| `quantity` | integer | Yes | Quantity to order. |
| `price` | number | No | Expected unit price. If it differs from the system price, the API may return a `price_mismatch` warning. |
| `stock_location_code` | string | No | Warehouse or stock location code. If omitted or invalid, the product's default stock location is used. |

### cURL Example

```bash
curl -X POST "https://example.com/rest/order/create" \
  -u "username:password" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": 65,
    "reference_code": "PO-2026-001",
    "shipping": {
      "firstname": "John",
      "lastname": "Doe",
      "company": "Acme BV",
      "street": "Keizersgracht",
      "house_number": "100",
      "postcode": "1015AA",
      "city": "Amsterdam",
      "country_iso2": "NL",
      "telephone": "+31201234567"
    },
    "invoice": {
      "firstname": "John",
      "lastname": "Doe",
      "company": "Acme BV",
      "street": "Herengracht",
      "house_number": "200",
      "postcode": "1016BS",
      "city": "Amsterdam",
      "country_iso2": "NL",
      "vat": "NL123456789B01"
    },
    "products": [
      {
        "sku": "LENOVO_0A36262",
        "quantity": 2,
        "price": 31.20
      },
      {
        "sku": "HP_1X644AA",
        "quantity": 1
      }
    ],
    "comment": "Deliver before Friday"
  }'
```

### PHP Example

```php
<?php
$url = 'https://example.com/rest/order/create';
$username = 'your_username';
$password = 'your_password';

$order = [
    'customer_id' => 65,
    'reference_code' => 'PO-2026-001',
    'shipping' => [
        'firstname' => 'John',
        'lastname' => 'Doe',
        'company' => 'Acme BV',
        'street' => 'Keizersgracht',
        'house_number' => '100',
        'postcode' => '1015AA',
        'city' => 'Amsterdam',
        'country_iso2' => 'NL',
    ],
    'invoice' => [
        'firstname' => 'John',
        'lastname' => 'Doe',
        'company' => 'Acme BV',
        'street' => 'Herengracht',
        'house_number' => '200',
        'postcode' => '1016BS',
        'city' => 'Amsterdam',
        'country_iso2' => 'NL',
        'vat' => 'NL123456789B01',
    ],
    'products' => [
        ['sku' => 'LENOVO_0A36262', 'quantity' => 2, 'price' => 31.20],
        ['sku' => 'HP_1X644AA', 'quantity' => 1],
    ],
    'comment' => 'Deliver before Friday',
];

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($order));
curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
curl_setopt($ch, CURLOPT_USERPWD, "$username:$password");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

$result = json_decode($response, true);

if (!empty($result['confirmed'])) {
    foreach ($result['data']['orders'] as $createdOrder) {
        echo "Order #{$createdOrder['order_id']} created. Status: {$createdOrder['status_name']}\n";

        foreach ($createdOrder['warnings'] ?? [] as $warning) {
            echo "Warning [{$warning['type']}] for SKU {$warning['sku']}\n";
        }
    }
} else {
    echo "Error: {$result['message']} ({$result['code']})\n";
}
```

### Successful Response

```json
{
  "confirmed": true,
  "message": "Order created successfully",
  "code": "ORDER_CREATED",
  "data": {
    "orders": [
      {
        "order_id": 336,
        "status_id": 1,
        "status_name": "New Order",
        "products_count": 2,
        "total_exc": 93.40,
        "total_inc": 113.01,
        "warnings": []
      }
    ]
  }
}
```

### Successful Response with Warnings

Depending on shop configuration, the API may return warnings when a submitted price does not match the system price, a requested product is out of stock, or the submitted stock location is invalid. The order is still created, but it may be placed on hold for manual review.

```json
{
  "confirmed": true,
  "message": "Order created with warnings",
  "code": "ORDER_CREATED_WITH_WARNINGS",
  "data": {
    "orders": [
      {
        "order_id": 337,
        "status_id": 7,
        "status_name": "On Hold",
        "products_count": 2,
        "total_exc": 93.40,
        "total_inc": 113.01,
        "warnings": [
          {
            "type": "price_mismatch",
            "sku": "LENOVO_0A36262",
            "submitted_price": 31.20,
            "batavi_price": 26.00
          },
          {
            "type": "out_of_stock",
            "sku": "HP_1X644AA",
            "available_stock": 0,
            "requested_quantity": 1
          }
        ]
      }
    ]
  }
}
```

### Split Order Response

Depending on shop configuration, products from different distributor regions may be split into separate Batavi orders.

```json
{
  "confirmed": true,
  "message": "Order split into 2 orders by distributor location",
  "code": "ORDER_CREATED_SPLIT",
  "data": {
    "orders": [
      {
        "order_id": 338,
        "status_id": 1,
        "status_name": "New Order",
        "products_count": 1,
        "total_exc": 62.40,
        "total_inc": 75.50,
        "distributor_country": "NL",
        "warnings": []
      },
      {
        "order_id": 339,
        "status_id": 1,
        "status_name": "New Order",
        "products_count": 1,
        "total_exc": 31.00,
        "total_inc": 37.51,
        "distributor_country": "DE",
        "warnings": []
      }
    ]
  }
}
```

### Unknown Product Error

If one or more SKUs or product IDs cannot be found, the order is rejected by default. Some shops may allow unknown products to be added as custom order lines; contact the shop administrator to confirm the configured behavior.

```json
{
  "error": true,
  "message": "Order rejected: unknown products",
  "code": "ORDER_REJECTED_UNKNOWN_PRODUCTS",
  "data": {
    "unknown_products": [
      "INVALID-SKU-1",
      "INVALID-SKU-2"
    ]
  }
}
```

When the order is rejected, fix or remove the values listed in `unknown_products` and retry the request.

---

## Get Order Details

Retrieves details for a single order. Use the `fields` query parameter to reduce response size when only specific fields are needed.

```http
GET /rest/order/list
```

### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `id` | integer | Yes | Order ID to retrieve. |
| `fields` | string | No | Comma-separated list of fields to return. |

### Available Fields

Order-level fields:

```text
id, referenceCode, status, currency, subtotalExc, subtotalInc,
shippingExc, shippingInc, totalExc, totalInc, datePurchased,
lastModified, storeType, customer, shipping, invoice, products
```

Product-level fields use the `products.` prefix:

```text
products.id, products.ordersId, products.status, products.sku,
products.name, products.model, products.mpn, products.gtin,
products.quantity, products.price, products.sales_price,
products.subtotalExclTax, products.subtotalInclTax, products.tax,
products.productVatPercentage, products.weight, products.brand,
products.stockLocation, products.updated
```

### cURL Examples

Get full order details:

```bash
curl -u "username:password" \
  "https://example.com/rest/order/list?id=336"
```

Get only high-level order status:

```bash
curl -u "username:password" \
  "https://example.com/rest/order/list?id=336&fields=id,referenceCode,status,lastModified"
```

Example response:

```json
{
  "confirmed": true,
  "data": [
    {
      "id": 336,
      "referenceCode": "PO-2026-001",
      "status": {
        "id": 2,
        "isOpen": 1,
        "isPaid": 0,
        "isShipped": 0,
        "name": "Processing"
      },
      "lastModified": "2026-04-10 14:30:00"
    }
  ]
}
```

Get order line statuses:

```bash
curl -u "username:password" \
  "https://example.com/rest/order/list?id=336&fields=id,status,products.id,products.sku,products.quantity,products.status"
```

Example response:

```json
{
  "confirmed": true,
  "data": [
    {
      "id": 336,
      "status": {
        "id": 2,
        "name": "Processing",
        "isOpen": 1,
        "isPaid": 0,
        "isShipped": 1
      },
      "products": [
        {
          "id": 5001,
          "sku": "LENOVO_0A36262",
          "quantity": 2,
          "status": {
            "id": 3,
            "name": "Shipped",
            "isOpen": 0,
            "isPaid": 0,
            "isShipped": 1
          }
        },
        {
          "id": 5002,
          "sku": "HP_1X644AA",
          "quantity": 1,
          "status": {
            "id": 1,
            "name": "New Order",
            "isOpen": 1,
            "isPaid": 0,
            "isShipped": 0
          }
        }
      ]
    }
  ]
}
```

Order status and product-line status are tracked independently. A product line can be shipped while other lines are still pending. Check `isShipped` on each product line to determine which items have been dispatched.

### PHP Example

```php
<?php
$orderId = 336;
$url = "https://example.com/rest/order/list?id=$orderId";

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_USERPWD, "$username:$password");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true);

if (!empty($result['confirmed'])) {
    foreach ($result['data'] as $order) {
        echo "Order #{$order['id']}\n";
        echo "Status: {$order['status']['id']} (shipped: {$order['status']['isShipped']})\n";
        echo "Total: {$order['currency']['code']} {$order['totalInc']}\n";

        foreach ($order['products'] as $product) {
            echo "- {$product['sku']} x{$product['quantity']} @ {$product['price']}";
            echo " status: {$product['status']['id']}\n";
        }
    }
}
```

### Full Response Example

```json
{
  "confirmed": true,
  "data": [
    {
      "id": 336,
      "status": {
        "id": 1,
        "cssClass": "status-new",
        "isOpen": 1,
        "isPaid": 0,
        "isShipped": 0,
        "isBlocking": 0,
        "isQuote": 0,
        "sortOrder": 1
      },
      "currency": {
        "id": 1,
        "title": "Euro",
        "code": "EUR",
        "value": 1.0
      },
      "referenceCode": "PO-2026-001",
      "subtotalExc": 93.40,
      "subtotalInc": 113.01,
      "shippingExc": 0.00,
      "shippingInc": 0.00,
      "totalExc": 93.40,
      "totalInc": 113.01,
      "datePurchased": "2026-04-10 10:58:00",
      "lastModified": "2026-04-10 10:58:00",
      "customer": {
        "id": 64,
        "firstname": "John",
        "lastname": "Doe",
        "company": "Acme BV",
        "emailAddress": "john@example.com"
      },
      "shipping": {
        "firstname": "John",
        "lastname": "Doe",
        "company": "Acme BV",
        "streetAddress": "Keizersgracht",
        "houseNumber": "100",
        "postcode": "1015AA",
        "city": "Amsterdam",
        "country": {
          "id": 150,
          "name": "Netherlands",
          "isoCode2": "NL"
        }
      },
      "products": [
        {
          "id": 5001,
          "ordersId": 336,
          "status": {
            "id": 1,
            "isOpen": 1,
            "isPaid": 0,
            "isShipped": 0
          },
          "sku": "LENOVO_0A36262",
          "name": "Lenovo ThinkPad Dock",
          "quantity": 2,
          "price": 31.20,
          "subtotalExclTax": 62.40,
          "subtotalInclTax": 75.50,
          "productVatPercentage": 21
        },
        {
          "id": 5002,
          "ordersId": 336,
          "status": {
            "id": 1,
            "isOpen": 1,
            "isPaid": 0,
            "isShipped": 0
          },
          "sku": "HP_1X644AA",
          "name": "HP Prelude Pro Backpack",
          "quantity": 1,
          "price": 31.00,
          "subtotalExclTax": 31.00,
          "subtotalInclTax": 37.51,
          "productVatPercentage": 21
        }
      ]
    }
  ]
}
```

## Response Field Reference

### Order Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Order ID. |
| `status` | object | Order status object. |
| `currency` | object | Currency details, including `code`, `title`, and `value`. |
| `referenceCode` | string | External reference code. |
| `subtotalExc` | number | Subtotal excluding VAT. |
| `subtotalInc` | number | Subtotal including VAT. |
| `shippingExc` | number | Shipping amount excluding VAT. |
| `shippingInc` | number | Shipping amount including VAT. |
| `totalExc` | number | Total excluding VAT. |
| `totalInc` | number | Total including VAT. |
| `datePurchased` | string | Order creation date. |
| `lastModified` | string | Last update date. |
| `customer` | object | Customer details. |
| `shipping` | object | Shipping address. |
| `invoice` | object | Invoice address. |
| `products` | array | Product lines. |

### Product Line Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Order line ID. |
| `ordersId` | integer | Parent order ID. |
| `status` | object | Product-line status object. |
| `sku` | string | Product SKU. |
| `name` | string | Product name. |
| `quantity` | integer | Ordered quantity. |
| `price` | number | Unit price. |
| `subtotalExclTax` | number | Line total excluding VAT. |
| `subtotalInclTax` | number | Line total including VAT. |
| `productVatPercentage` | integer | VAT percentage. |
| `brand` | object | Brand details, including `id` and `name`. |
| `stockLocation` | object | Warehouse details, including `id`, `code`, and `name`. |
| `model` | string | Product model. |
| `mpn` | string | Manufacturer part number. |
| `gtin` | string | Global Trade Item Number. |

### Status Object

The status object is used at both order level and product-line level.

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Status ID. |
| `name` | string | Public status name, such as `New Order`, `Processing`, `Shipped`, or `Delivered`. |
| `isOpen` | integer | `1` means the order or line is still open. |
| `isPaid` | integer | `1` means the order or line is marked as paid. |
| `isShipped` | integer | `1` means the order or line is marked as shipped. |
| `isBlocking` | integer | `1` means the status blocks further processing. |
| `isQuote` | integer | `1` means the record is a quote, not a confirmed order. |

## Response Codes

### Success Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `ORDER_CREATED` | `200` | Order created successfully without warnings. |
| `ORDER_CREATED_WITH_WARNINGS` | `200` | Order created with warnings and may be placed on hold. |
| `ORDER_CREATED_SPLIT` | `200` | Order split into multiple orders by distributor region. |

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | `401` | Invalid or missing authentication credentials. |
| `METHOD_NOT_ALLOWED` | `405` | Incorrect HTTP method. For example, `GET` was used for order creation. |
| `INCORRECT_PARAMS` | `400` | Missing or invalid required parameters. |
| `CUSTOMER_NOT_FOUND` | `400` | The supplied `customer_id` does not match a known login. |
| `ORDER_REJECTED_UNKNOWN_PRODUCTS` | `400` | One or more SKUs or product IDs were not found. |
| `ERROR_WHILE_TRYING_TO_EXECUTE_OPERATION` | `500` | Server error while saving the order. |

### Warning Types

Warnings are returned in the `warnings` array.

| Type | Fields | Description |
|------|--------|-------------|
| `price_mismatch` | `sku`, `submitted_price`, `batavi_price` | Submitted unit price differs from the system price for the customer. |
| `out_of_stock` | `sku`, `available_stock`, `requested_quantity` | Requested quantity exceeds available stock. |
| `invalid_stock_location` | `sku`, `submitted_stock_location` | Submitted stock location code was not found; the product's default stock location was used. |

Whether warnings are generated and how they affect order status depends on shop configuration. When warning checks are enabled, orders with warnings are typically placed on hold for manual review.

## Implementation Notes

- `customer_id` is the login ID supplied by the shop administrator.
- Always send both `shipping` and `invoice`, even when both addresses are the same.
- Duplicate addresses are detected automatically and do not create duplicate address records.
- If `price` is omitted from a product line, Batavi calculates the price automatically.
- If `price` is submitted and differs from the Batavi price, the API may return a `price_mismatch` warning.
- Use the `fields` parameter on `/rest/order/list` when a connector only needs a small subset of order data.
- Treat `confirmed: true` as a successful operation. Treat `error: true` as a failed operation.
