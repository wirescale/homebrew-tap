# homebrew-wirescale

Homebrew tap для [wirescale](https://github.com/wirescale) — hub-and-spoke VPN
(демон `wirescaled` + CLI `wirescale`). Prebuilt bottle: бинарники для
macOS amd64/arm64 раздаются с `https://tap.wirescale.org/`.

## Установка

```sh
brew tap wirescale/wirescale
brew install wirescale
```

Демон требует root (TUN, маршруты, pf) — запусти его как Homebrew-сервис:

```sh
sudo brew services start wirescale
```

(юнит генерирует Homebrew в `/Library/LaunchDaemons/sh.brew.wirescale.plist`;
`sudo brew services stop|restart wirescale` — остановка/перезапуск)

`sudo brew services start` chown'ит keg в `root:admin` + sticky (чтобы юзер не
подменил бинарь root-сервиса). Демон запускается через обёртку
`wirescale-service-wrap`: при каждом старте она возвращает владение твоему
пользователю, поэтому `brew upgrade`/`brew uninstall` работают без `sudo rm`.

## Обновление

```sh
brew update
brew upgrade wirescale
sudo brew services restart wirescale   # перезапустить демон новой версии
```

## Удаление

Homebrew **не останавливает сервисы при `brew uninstall`** — но на macOS демон
сам следит за своим бинарём: после удаления keg он завершается и снимает
launchd-юнит в течение ≤30 секунд. Для мгновенной остановки:

```sh
sudo brew services stop wirescale
brew uninstall wirescale
```

Старый ручной юнит (`wirescaled service install`, label `wirescale`) Homebrew не
видит. При переходе на brew services сними его:

```sh
sudo wirescaled service remove
```

## GUI: wirescale-ui (формула)

Menu-bar клиент (Tauri v2 + Svelte 5), только Apple Silicon. Требует демон
`wirescale` (тянется автоматически через `depends_on`).

```sh
brew install wirescale-ui   # приложение, открыть: wirescale-ui
```

`.app` кладётся в prefix Homebrew (`$(brew --prefix)/opt/wirescale-ui/…`), запуск —
команда `wirescale-ui`. Не переноси `.app` в `/Applications` руками: `brew upgrade`
обновляет копию в prefix, а перенесённая останется старой.

Формула, а не cask: Homebrew сам ставит `com.apple.quarantine` на cask-загрузки, а
подписи Developer ID у нас нет (нет Apple Developer Program) — cask установил бы
«повреждённое» приложение. Формулы карантином не помечаются. Подробности —
`release/tap/update-gui-formula.sh`.

Формула появляется в тапе после первого прогона `release/ci/publish-tap.sh`
(генератор: `release/tap/update-gui-formula.sh <ver>`).

## GUI без Homebrew

Тот же `.app` ставится в `~/Applications` curl-инсталлером с
`https://tap.wirescale.org/` (файл обновляется на каждом релизе, версия и
sha256 зашиты в него):

```sh
curl -fsSL https://tap.wirescale.org/install-wirescale-ui.sh | bash
```

curl не ставит карантин, поэтому Gatekeeper приложение не блокирует. Инсталлер
перезаписывает `~/Applications/Wirescale.app` (закрой приложение перед запуском).
Если ты скачивал `.app` браузером и macOS ругается на «повреждённое» приложение —
сними карантин: `xattr -dr com.apple.quarantine ~/Applications/Wirescale.app`.


## Примечания

- macOS-агент — только leaf/spoke (`wirescale peers join <TOKEN>`);
  hub/observer — Linux-only.
- Дефолтные пути демона — `/usr/local/var/wirescaled` (данные),
  `/usr/local/etc/wirescale.conf` (конфиг),
  `/Library/LaunchDaemons/sh.brew.wirescale.plist` (юнит Homebrew-сервиса);
  сокет LocalControl — `/var/run/wirescaled.sock` (macOS и Linux);
  не зависят от prefix Homebrew (`/opt/homebrew` на Apple Silicon).
- Версия формулы может отличаться от версии бинарей (`wirescale --version`):
  ревизия сборки кодируется 4-м компонентом (0.0.2.1 > 0.0.2) — `brew upgrade`
  подхватывает её без бампа версии приложения.
- Формула обновляется на сборочной машине: `release/tap/update-formula.sh <ver>`
  (в репозитории release), GUI-формула — `release/tap/update-gui-formula.sh <ver>`.
- Внутри `.app` версия = semver из `VERSION` (например 0.1.0), а ревизия сборки
  лежит в Finder Get Info (Build) — так же, как `wirescale --version`.