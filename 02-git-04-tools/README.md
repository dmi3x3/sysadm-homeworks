# Домашнее задание к занятию «2.4. Инструменты Git»
## 1. Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.

    dmitriy@dellix:~/netology/terraform$ git show aefea
    commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
    Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>
    Date:   Thu Jun 18 10:29:58 2020 -0400

        Update CHANGELOG.md

    diff --git a/CHANGELOG.md b/CHANGELOG.md
    index 86d70e3e0..588d807b1 100644
    --- a/CHANGELOG.md
    +++ b/CHANGELOG.md
    @@ -27,6 +27,7 @@ BUG FIXES:
     * backend/s3: Prefer AWS shared configuration over EC2 metadata credentials by default ([#25134](https://github.com/hashicorp/terraform/issues/25134))
     * backend/s3: Prefer ECS credentials over EC2 metadata credentials by default ([#25134](https://github.com/hashicorp/terraform/issues/25134))
     * backend/s3: Remove hardcoded AWS Provider messaging ([#25134](https://github.com/hashicorp/terraform/issues/25134))
    +* command: Fix bug with global `-v`/`-version`/`--version` flags introduced in 0.13.0beta2 [GH-25277]
     * command/0.13upgrade: Fix `0.13upgrade` usage help text to include options ([#25127](https://github.com/hashicorp/terraform/issues/25127))
     * command/0.13upgrade: Do not add source for builtin provider ([#25215](https://github.com/hashicorp/terraform/issues/25215))
     * command/apply: Fix bug which caused Terraform to silently exit on Windows when using absolute plan path ([#25233](https://github.com/hashicorp/terraform/issues/25233))

### или вот так:

    dmitriy@dellix:~/netology/terraform$ git log -p -1 aefea | sed -n '1p; 5p'
    commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
        Update CHANGELOG.md


## 2. Какому тегу соответствует коммит 85024d3?

    dmitriy@dellix:~/netology/terraform$ git log -p -1 85024d3 | sed -n '5p'
        v0.12.23

## 3. Сколько родителей у коммита b8d720? Напишите их хеши.

    dmitriy@dellix:~/netology/terraform$ git show -s --format=%p b8d720
    56cd7859e 9ea88f22f

## 4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

    dmitriy@dellix:~/netology/terraform$ git log v0.12.23..v0.12.24 --oneline
    33ff1c03b (tag: v0.12.24) v0.12.24
    b14b74c49 [Website] vmc provider links 
    3f235065b Update CHANGELOG.md
    6ae64e247 registry: Fix panic when server is unreachable
    5c619ca1b website: Remove links to the getting started guide's old location
    06275647e Update CHANGELOG.md
    d5f9411f5 command: Fix bug when using terraform login on Windows
    4b6d06cc5 Update CHANGELOG.md
    dd01a3507 Update CHANGELOG.md
    225466bc3 Cleanup after v0.12.23 release

## 5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).

    dmitriy@dellix:~/netology/terraform$ git log -S 'func providerSource' --oneline
    5af1e6234 main: Honor explicit provider_installation CLI config when present
    8c928e835 main: Consult local directories as potential mirrors of providers
    dmitriy@dellix:~/netology/terraform$ git show 5af1e6234 | grep providerSource
    -	providerSrc := providerSource(services)
    +	providerSrc, diags := providerSource(config.ProviderInstallation, services)
     // providerSource constructs a provider source based on a combination of the
    -func providerSource(services *disco.Disco) getproviders.Source {
    +func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {
    +		source, moreDiags := providerSourceForCLIConfigLocation(sourceConfig.Location, services)
    +func providerSourceForCLIConfigLocation(loc cliconfig.ProviderInstallationSourceLocation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {

    dmitriy@dellix:~/netology/terraform$ git show `8c928e835` | grep providerSource
    + providerSrc := providerSource(services)
    +// providerSource constructs a provider source based on a combination of the
    +func providerSource(services *disco.Disco) getproviders.Source {

### Ответ: 8c928e835

## 6. Найдите все коммиты в которых была изменена функция globalPluginDirs.

    dmitriy@dellix:~/netology/terraform$ git log -L :'func globalPluginDirs':plugins.go | grep "^commit "
    commit 78b12205587fe839f10d946ea3fdc06719decb05
    commit 52dbf94834cb970b510f2fba853a5b49ad9b1a46
    commit 41ab0aef7a0fe030e84018973a64135b11abcd70
    commit 66ebff90cdfaa6938f26f908c7ebad8d547fea17
    commit 8364383c359a6b738a436d1b7745ccdce178df47


## 7. Кто автор функции synchronizedWriters?

    dmitriy@dellix:~/netology/terraform$ git log -S 'func synchronizedWriters'
    commit bdfea50cc85161dea41be0fe3381fd98731ff786
    Author: James Bardin <j.bardin@gmail.com>
    Date:   Mon Nov 30 18:02:04 2020 -0500

    remove unused

    commit 5ac311e2a91e381e2f52234668b49ba670aa0fe5
    Author: Martin Atkins <mart@degeneration.co.uk>
    Date:   Wed May 3 16:25:41 2017 -0700

    main: synchronize writes to VT100-faker on Windows
    
    We use a third-party library "colorable" to translate VT100 color
    sequences into Windows console attribute-setting calls when Terraform is
    running on Windows.
    
    colorable is not concurrency-safe for multiple writes to the same console,
    because it writes to the console one character at a time and so two
    concurrent writers get their characters interleaved, creating unreadable
    garble.
    
    Here we wrap around it a synchronization mechanism to ensure that there
    can be only one Write call outstanding across both stderr and stdout,
    mimicking the usual behavior we expect (when stderr/stdout are a normal
    file handle) of each Write being completed atomically.

### Ответ: Martin Atkins