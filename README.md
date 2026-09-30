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
sudo wirescaled service install
```

(юнит `/Library/LaunchDaemons/wirescale.plist` и его запуск `service install` выполняет сам)

## Обновление

```sh
brew update
brew upgrade wirescale
```

## GUI: wirescale-ui (cask)

Menu-bar клиент (Tauri v2 + Svelte 5). Требует установленной формулы `wirescale`
(AC-UI-13: cask тянет формулу автоматически через `depends_on`).

```sh
brew install --cask wirescale-ui
```

Caveats:

- Cask пока **не опубликован**: `.app` собирается на macOS (`tauri build --bundles app,dmg`),
  тарболлы и sha256 появятся после вехи B3. Генератор: `release/tap/update-cask.sh <ver>`.
- Приложение подписано ad-hoc (Gatekeeper): первый запуск — правой кнопкой →
  «Открыть», затем подтвердить. Полная notarization — отдельная веха.
- Демон запускается через службу: `sudo wirescaled service install`.

## Примечания

- macOS-агент — только leaf/spoke (`wirescale peers join <TOKEN>`);
  hub/observer — Linux-only.
- Дефолтные пути демона — `/usr/local/var/wirescaled` (данные),
  `/usr/local/etc/wirescale.conf` (конфиг), `/Library/LaunchDaemons` (юнит);
  не зависят от prefix Homebrew (`/opt/homebrew` на Apple Silicon).
- Версия формулы может отличаться от версии бинарей (`wirescale --version`):
  ревизия сборки кодируется 4-м компонентом (0.0.2.1 > 0.0.2) — `brew upgrade`
  подхватывает её без бампа версии приложения.
- Формула обновляется на сборочной машине: `release/tap/update-formula.sh <ver>`
  (в репозитории release).