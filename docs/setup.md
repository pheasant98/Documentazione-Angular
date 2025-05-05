# Setup

Per installare e usare Angular 7 o 12 occore prima di tutto installare il gestore di package npm:

- [Download Node 10.24.1](https://nodejs.org/en/blog/release/v10.24.1) per Angular 7<br>
- [Download Node 12.22.7](https://nodejs.org/en/blog/release/v12.22.7) per Angular 12

!!!Info
    Per gestire le versioni di node è utile installare NVM. <br>
    Scarica [nvm-setup.exe](https://github.com/coreybutler/nvm-windows/releases) per installare npm.<br>
    Verifica con:
    
    - `nvm version` per verificare se NVM è stato installato correttamente.
    - `npm -v` per verificare se npm è stato installato correttamente.

Si passa poi a installare la Angular CLI:

`npm install -g @angular/cli`

o

`npm install -g @angular/cli@12`

e verificare con il commando `ng version` per verificare la corretta installazione.

Per creare un progetto Angular si utilizza:

`ng new nome-progetto`

e per avviare il progetto: 

- spostiamoci nella directory con `cd nome-progetto`, 
- `ng serve` per lanciare il server Angular,
- collegarsi al http://localhost:4200 nel browser.

