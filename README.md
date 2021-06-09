```sh
$ export GITHUB_TOKEN=секрет
$ export GITHUB_USERNAME=Lam0Rka
$ export PACKAGE_MANAGER=apt
$ export GPG_PACKAGE_NAME=gpg #т.к дальше по туториалу будет использоваться именно он
```

```sh
$ $PACKAGE_MANAGER install xclip #Устанавливаем утилиту xclip, предоставляющую доступ к буферу обмена Х из коммандной строки
$ alias gsed=sed
$ alias pbcopy='xclip -selection clipboard'
$ alias pbpaste='xclip -selection clipboard -o'
```

```sh
# Скачивание и установка пакета Go, для работы с релизами Github
$ cd ${GITHUB_USERNAME}/workspace
$ pushd . 
$ source scripts/activate
$ go get github.com/aktau/github-release
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab888 projects/lab999
$ cd projects/lab999
#подготовили репозиторий для последующей работы и создали катало lab999
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab999
```

```sh
#Заменили все упоминания lab888 на lab999
$ gsed -i 's/lab08/lab09/g' README.md
```

В этом блоке мы устанавливаем GPG для шифровании информации(текста например) и для электронных цифровых подписей.
Затем проверяем если ли у нас секретные ключи и потом сами генерируем это ключи под параметры которые нам предложит GPG
```sh
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}
$ gpg --list-secret-keys --keyid-format LONG
$ gpg --full-generate-key
#Вводим секретный ключ
$ gpg --list-secret-keys --keyid-format LONG
$ gpg -K ${GITHUB_USERNAME}
# Сохраняем перменную с публичным ключом
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Сохраняем переменную с секретным ключом
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Выводим ключ в ASCII и копируем его в буфер обмена
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy
$ pbpaste
# Зесь мы должны открыть гитхаб сеттинг, и устанавливаем ключ, но у меня open не работает и поэтому я открыл не через консоль
$ open https://github.com/settings/keys
#Добавляем к гитхаб ключ который мы создали
$ git config user.signingkey ${GPG_SEC_KEY_ID}
$ git config gpg.program gpg
```

```sh
# Настраиваем скрипт для добавления сообщения к тегу
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```
Логинимся в трэвис но у меня не работае  --auto поэтому всякий раз использу свой трэвис логин
```sh
$ travis login --github-token (token)
$ travis enable
```

```sh
# Создание тега с сообщением с информацией а затем вервефеируем этот тег, тут у меня возникла идея написать сразу -s -v, но эксперементировать я не стал и дальше пошёл по туториалу
$ git tag -s v0.1.0.0
$ git tag -v v0.1.0.0
# Смотрим на изменения
$ git show v0.1.0.0
# Пушим
$ git push origin main --tags
```
Затем здесь делаем релиз, НО у меня не работала эта команда потому что у меня не скачался go, но вскоре проблема была устранена и всё удачно заработало(к слову у меня не рботала 3-я команда, но после того как я создал новый токен и прописал его, всё заработало)
```sh
$ github-release --version
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab999 \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```

```sh
# Добавление артефакта с указанием ОС и архитектуры, на которых происходила компиляция библиотек
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m` 
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab999 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab999

# Скачивание артефакта из раздела релизов для проверки
$ wget https://github.com/${GITHUB_USERNAME}/lab999/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
#Проверяем что за архив мы скачали
$ tar -ztf ${PACKAGE_FILENAME}
```

