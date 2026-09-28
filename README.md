# homebrew-wirescale

Homebrew tap для [wirescale](https://github.com/wirescale) — hub-and-spoke VPN
(демон `wirescaled` + CLI `wirescale`). Prebuilt bottle: бинарники для
macOS amd64/arm64 раздаются с `https://tap.wirescale.org/`.

## Установка

```sh
brew tap wirescale/wirescale
brew install wirescale
```

Демон требует root (TUN, маршруты, pf) — после установки:

```sh
sudo wirescale service install
sudo launchctl load /Library/LaunchDaemons/wirescale.plist
```

## Обновление

```sh
brew update
brew upgrade wirescale
```

## Примечания

- macOS-агент — только leaf/spoke (`wirescale peers join <TOKEN>`);
  hub/observer — Linux-only.
- Дефолтные пути демона — `/usr/local/var/wirescaled` (данные),
  `/usr/local/etc/wirescale.conf` (конфиг), `/Library/LaunchDaemons` (юнит);
  не зависят от prefix Homebrew (`/opt/homebrew` на Apple Silicon).
- Формула обновляется на сборочной машине: `deploy/tap/update-formula.sh <ver>`
  (в репозитории wirescale).