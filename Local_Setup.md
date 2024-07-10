# Setting Up Primero Locally
The instructions in this guide assume that you are running MacOS, if you are running another OS you will need to adapt the instructions accordingly.

## Cloning the Primero Repository

To get a fresh copy of the code, you need to have [git](http://git-scm.com/book/en/v2/Git-Internals-Packfiles) installed.

```bash
brew update
brew install git
```

Navigate to a directory in your shell where you would like to download Primero, and clone the repository.

```bash
git clone https://github.com/primeroIMS/primero.git
```

The `main` branch which has the latest releasse is the default. Please verify that you are on the `main` branch. Below commands will switch to main.

```bash
cd primero
git checkout main
```

# Installing Dependencies
## Installing Ruby using `rbenv`

`rbenv` is a version manager and installer for ruby runtimes. It allows you to install the correct version of ruby for each project that you are working on.

In order to install ruby using `rbenv`, you will need to install the following dependencies, which are used to build ruby from source.

```bash
sudo brew install curl g++ gnupg2 gcc autoconf automake bison libc6-dev libffi-dev libgdbm-dev libncurses5-dev libsqlite3-dev libtool libyaml-dev make pkg-config sqlite3 zlib1g-dev libgmp-dev  libreadline-dev libssl-dev
```

You will then need to install `rbenv` as well as `ruby-build`. There is a convenience script provided by `rbenv` that will help to install them.

```bash
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
```

The convenience script does not modify your `.bashrc` file. You need to update it manually with the snippet below. If you use another shell such as zsh, you will need to edit the correct startup file.

```bash
echo 'export PATH="$PATH:~/.rbenv/bin"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

In the Primero top-level directory, there is a file `.ruby-version`, which contains the correct version of ruby to install. Install the correct version as follows:

```bash
cat .ruby-version
# This will print something like: ruby-3.2.3
# rbenv needs the version number, but not the ruby- prefix.
rbenv install 3.2.3 # replace 3.2.3 with whatever version is in .ruby-version
```

It will take several minutes to build and install ruby, depending on the speed of your machine.
Once you have succeeded in installing ruby, it is worth checking that you are now using the right version.

```bash
ruby --version
# This should print something like: ruby 3.2.3 (or whatever the current version in the .ruby-version is)
```

## Installing node using `nvm`

`nvm` is a version manager for node runtimes. It is similar to rbenv, but manages node rather than ruby.

`nvm` has a convenience script which will install it automatically. It should also automatically update your shell startup file.

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

In order to start using `nvm`, you will need to close and reopen your terminal.

To install the version of node needed by primero, run the following command:
```shell
nvm install --lts
```

```
## Building the Containers

You will need to have `docker` and `docker-compose` installed locally.
Make sure that you are at the root of the primero project.

```bash
cd ./docker
sudo ./build.sh postgres solr
```
## Running the Containers

You should run this from inside the docker directory.
```bash
sudo ./compose.local.sh up -d postgres
sudo ./compose.local.sh run solr make-primero-core.sh primero-test
sudo ./compose.local.sh up -d solr
```

# Configuring Primero for Local Development

Primero is partially configured with a number of yaml files. There are example versions of these files provided for local development. They need to be copied to the correct locations in order for Primero to function.

Execute these from the root directory of the repository. You may want to review these files and potentially make changes to them once they have been copied.
```bash
cp config/database.yml.development config/database.yml
cp config/locales.yml.development config/locales.yml
cp config/mailers.yml.development config/mailers.yml
cp config/sunspot.yml.development config/sunspot.yml
mkdir log
```

You will also need to install some system-wide dependencies required to build and run Primero.

```bash
sudo brew install libpq imagemagick libsodium p7zip
```

Execute the following while still on the project root to install Primero's ruby and node dependencies:

```bash
bundle install
npm ci
```

You now need to create the two Postgres databases (the default database, and a database used for unit testing).

```shell
rails db:create
rails db:migrate
rails db:seed

RAILS_ENV=test rails db:migrate
```

You also need to generate the translation files.

```shell
rails primero:i18n_js
```

Finally, you need to set a number of environment variables which contain necessary secrets.

The environment variables are:
- PRIMERO_SECRET_KEY_BASE
- DEVISE_SECRET_KEY
- DEVISE_JWT_SECRET_KEY

The following bash command will generate a secure secret for each of these and add them to your `~/.bashrc`:

```bash
for v in PRIMERO_SECRET_KEY_BASE DEVISE_SECRET_KEY DEVISE_JWT_SECRET_KEY;
do echo "export ${v}=$(openssl rand -hex 16)" >> ~/.bashrc;
done;
```

If you use a different shell, for example zsh, you will need to add these to the correct startup file.

# Running Primero Locally

For this, you will want to create two different terminal tabs/windows. In one, run the following, which will build and serve the frontend application:

```shell
npm run dev
```

In the other window, run the following command, which will run the rails server that hosts the Primero backend.

```bash
rails s
```

Visit http://localhost:3000/ in your browser, and you should see the following page. Note that it can take a while to load for the very first time, as the frontend application bundle needs to be compiled. Because of this, it is easiest to leave the `npm` process running in the background if you are only making changes to the backend code.

Log in using the default credentials `primero/primer0!`.


