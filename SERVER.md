# The New E-Mail Protocol (NEMP) - server documentation

## Server parts

- SQLite database
- HTTP server (just for redirecting to HTTPS server)
- HTTPS server (running admin API, client API, web admin and web mail (web client)

## Used dependencies
- Node
- NPM
- NPM dependencies listed in [**package.json**](./src/package.json)

## Server settings

Server settings is stored in **settings.json** file. This file can be auto-generated by this command:

```console
node index.js --create-settings
```

- **webadmin_run** - describes whether to run web admin on web server (true / false)
- **webadmin_url** - describes web server URL of web admin
- **webadmin_path** - the absolute or relative path to web admin files
- **webadmin_ttl** - web admin token expiration time (in seconds)
- **webmail_run** - describes whether to run web mail on web server (true / false)
- **webmail_url** - describes web server URL of web mail
- **webmail_path** - the absolute or relative path to web mail files
- **console_run** - describes whether to run dev console on web server (true / false)
- **console_url** - describes web server URL of dev console
- **console_path** - the absolute or relative path to dev console files
- **http_port** - HTTP server port (for redirect to HTTPS only)
- **https_port** - HTTPS server port
- **https_cert_path** - file system path to HTTPS certificates (**privkey.pem**, **cert.pem** and **chain.pem**)
- **log_to_file** - whether server should log into log file (true / false) - if set to false, it will log to console only
- **db_file** - server database file name (in SQL format) - if it doesn't exist, it is auto-generated on start
- **log_file** - server log file name

## Web server
Web server serves for WebSocket mail protocol, Web Admin and Web Mail.
HTTPS is mandatory. HTTP serves only for redirecting requests to HTTPS.

## Server database

By default the server database is using SQLite file database.
Passwords are stored as Argon2 hash. These hashes are used in **users** and **admins** tables.

This database consists of the following tables:

### Database structure

#### admins
- id INTEGER PRIMARY KEY AUTOINCREMENT
- user VARCHAR(32) NOT NULL UNIQUE
- pass VARCHAR(255) NOT NULL
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP

#### admins_login
- id INTEGER PRIMARY KEY AUTOINCREMENT
- id_admin INTEGER
- token VARCHAR(64) NOT NULL UNIQUE
- updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (id_admin) REFERENCES admins(id)

#### domains
- id INTEGER PRIMARY KEY AUTOINCREMENT
- name VARCHAR(255) NOT NULL UNIQUE
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP

#### users
- id INTEGER PRIMARY KEY AUTOINCREMENT
- id_domain INTEGER
- name VARCHAR(64) NOT NULL
- visible_name VARCHAR(255) NULL
- pass VARCHAR(255) NOT NULL
- photo VARCHAR(255) NULL UNIQUE
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (id_domain) REFERENCES domains(id)

#### aliases
- id INTEGER PRIMARY KEY AUTOINCREMENT
- alias VARCHAR(64) NOT NULL
- id_domain INTEGER
- mail VARCHAR(255) NOT NULL
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (id_domain) REFERENCES domains(id)

#### messages
- id INTEGER PRIMARY KEY AUTOINCREMENT
- id_user INTEGER
- email VARCHAR(255) NOT NULL
- message TEXT NOT NULL
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (id_user) REFERENCES users(id)

#### contacts
- id INTEGER PRIMARY KEY AUTOINCREMENT
- id_user INTEGER
- name VARCHAR(64) NOT NULL
- visible_name VARCHAR(255)
- email VARCHAR(255) NOT NULL UNIQUE
- created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (id_user) REFERENCES users(id)
