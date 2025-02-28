# Calculate Taxes Manually in Checkout

In this document, you’ll learn how to manually calculate taxes during checkout if you have automatic tax calculation disabled in a region.

## Overview

By default, taxes are automatically calculated by Medusa during checkout. This behavior can be disabled for a region using the Admin APIs or the Medusa admin to limit the requests being sent to a tax provider.

If you disable this behavior, you must manually trigger taxes calculation. When taxes are calculated, this means that requests will be sent to the tax provider to retrieve the tax rates.

## How to Manually Calculate Taxes in Checkout?

This section explores different ways you can calculate taxes based on your purpose.

### Use Calculate Cart Taxes Endpoint

The [Calculate Cart Taxes](https://docs.medusajs.com/api/store/#tag/Cart/operation/PostCartsCartTaxes) endpoint forces the calculation of taxes for a cart during checkout. This bypasses the option set in admin to not calculate taxes automatically, which results in sending requests to the tax provider.

This calculates and retrieves the taxes on the cart and each of the line items in that cart.

### Use CartService

The `CartService` class has a method `retrieve` that gets a cart by ID. In that method, taxes are calculated only if automatic taxes calculation is enabled for the region the cart belongs to.

You can, however, force calculating the taxes of the cart by passing in the third parameter an option containing the key `force_taxes` with its value set to `true`.

For example:

```jsx
cartService.retrieve('cart_01G8Z...', { }, { force_taxes: true });
```

:::tip

You can learn how to [retrieve and use services](../services/create-service.md#using-your-custom-service) in this documentation.

:::

### Use decorateLineItemsWithTotals function in Endpoints

If you want to calculate the taxes of line items on the storefront in an endpoint and return the necessary fields in the result, you can use the `decorateLineItemsWithTotals` method in your endpoint:

```jsx
//at the beginning of the file
import { decorateLineItemsWithTotals } from "@medusajs/medusa/dist/api/routes/store/carts/decorate-line-items-with-totals"

export default () => {
  //...

  router.get("/store/line-taxes", async (req, res) => {
    //example of retrieving cart
    const cartService = req.scope.resolve("cartService");
    const cart = await cartService.retrieve(cart_id)
    
    //...
    //retrieve taxes of line items
    const data = await decorateLineItemsWithTotals(cart, req, {
      force_taxes: true
    });
    
    return res.status(200).json({ cart: data });
  })
}
```

The `decorateLineItemsWithTotals` method accepts an options object as a third parameter. If you set `force_taxes` to `true` in that object, the totals of the line items can be retrieved regardless of whether the automatic calculation is enabled in the line item’s region.

### Use TotalsService

You can calculate and retrieve taxes of line items using the `getLineItemTotals` method available in the `TotalService` class. All you need to do is pass in the third argument to that method an options object with the key `include_tax` set to true:

```jsx
const itemTotals = await totalsService
  .getLineItemTotals(item, cart, {
    include_tax: true
  })
```

:::tip

You can learn how to [retrieve and use services](../services/create-service.md#using-your-custom-service) in this documentation.

:::

## What’s Next 🚀

- Learn about [tax-inclusive pricing](inclusive-pricing.md).
- Learn about available methods in [CartsService](../../../references/services/classes/CartService.md) and [TotalsService](../../../references/services/classes/TotalsService.md).
