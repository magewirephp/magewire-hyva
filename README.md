# Magewire Compatibility with Hyvä

[![Mago](https://github.com/magewirephp/magewire-hyva/actions/workflows/mago.yml/badge.svg?branch=main)](https://github.com/magewirephp/magewire-hyva/actions/workflows/mago.yml)

> Makes [Magewire](https://github.com/magewirephp/magewire) feel native to the Hyvä theme — so reactive, server-driven components just work, without writing JavaScript.

This is the compatibility layer between [Magewire](https://github.com/magewirephp/magewire) v3 and the [Hyvä](https://hyva.io) frontend. It wires Magewire into the Hyvä asset pipeline, hands AlpineJS off to Magewire, bridges Magewire's flash messages to the Hyvä notifier, and keeps Hyvä Checkout components built for Magewire v1 working on v3.

## What it does

### Tailwind / asset registration

Magewire and this module ship frontend assets (AlpineJS plugins, Tailwind sources). On the `hyva_config_generate_before` event the module registers both module paths as Hyvä [extensions](https://docs.hyva.io/hyva-themes/compatibility-modules/tailwind-config-merging.html), so their styles and scripts are picked up by the Hyvä Tailwind build automatically.

### AlpineJS consolidation

Magewire already bundles AlpineJS. To avoid loading Alpine twice, the module disables Hyvä's standalone `script-alpine-js` and moves it under Magewire's loader, guaranteeing Alpine is initialized **before** Magewire boots.

### Flash messages

Bridges the Magewire `magewire:flash-messages:dispatch` event to Hyvä's message/notifier UI, so server-side flash messages dispatched from a component surface through the theme's native notification component.

### Hyvä Checkout backwards compatibility

Hyvä Checkout was built on Magewire v1 (Livewire v2). Magewire v3 (Livewire v3) changed the `wire:` directive and `entangle` semantics:

| Magewire v1 (Livewire v2) | Magewire v3 (Livewire v3)      |
|---------------------------|--------------------------------|
| `wire:model` (instant)    | `wire:model.live`              |
| `wire:model.defer`        | `wire:model` (now the default) |
| `wire:model.lazy`         | `wire:model.blur`              |
| `$wire.entangle()` (live) | `$wire.entangle()` (deferred)  |

The `SupportHyvaCheckoutBackwardsCompatibility` Magewire feature flags components that need the old behavior by pushing a `bc.enabled` memo into the snapshot. The flag is resolved, in priority order, from:

1. The `#[HandleBackwardsCompatibility]` attribute on the component class
2. A previously hydrated value from the component's data store
3. Whether the component lives inside the `hyva-checkout-main` layout container

Frontend JS then reads the flag to migrate `wire:model` directives and make `entangle` default to live — so existing Hyvä Checkout components run on Magewire v3 unchanged.

## Requirements

- `magewirephp/magewire` `>=3.2`
- `Hyva_Theme`
- `Hyva_Checkout` (for the checkout backwards-compatibility feature)

The module declares a `sequence` after `Magewirephp_Magewire`, `Hyva_Theme`, and `Hyva_Checkout`.

## Installation

```bash
composer require hyva-themes/magento2-hyva-checkout
bin/magento module:enable Magewirephp_MagewireCompatibilityWithHyva
bin/magento setup:upgrade
```

Rebuild the Hyvä theme so the merged Tailwind config and assets are picked up:

```bash
cd app/design/frontend/<Vendor>/<theme>/web/tailwind
npm run build
```

## Documentation

See the [Hyvä docs](https://docs.hyva.io/) and the [Magewire docs](https://github.com/magewirephp/magewire).

## Security Vulnerabilities

Please do not report security issues publicly. Disclose them privately to the Magewire maintainers.

## License

Open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
