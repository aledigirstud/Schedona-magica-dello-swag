# Schedona magica dello swag

## Obiettivo

Ricostruire in KiCad la scheda CUAV X7 descritta nei PDF [BASE](docs/X7_BASE_RC03.pdf), [CORE](docs/X7%2BCORE_RC03.pdf) e [IMU](docs/X7_IMU_RC03.pdf), adattandola alle nostre esigenze: **dimensioni contenute, semplicità circuitale e di assemblaggio, compatibilità con ArduSub**.

L’obiettivo è riunire alimentazione, microcontrollore, sensori e interfacce su **un unico PCB**, eliminando la struttura originale in tre elementi separati. I PDF costituiscono il riferimento da cui ricostruire le funzioni utili; dimensioni finali, numero di propulsori e periferiche richieste devono ancora essere definiti.

La compatibilità con ArduSub è un requisito da verificare sul prototipo. Se cambiano pin, sensori o periferiche rispetto alla scheda originale, occorre adattare e collaudare la definizione hardware e il bootloader: il firmware della X7 non va considerato automaticamente compatibile. Il percorso di riferimento è il [porting ufficiale ArduPilot](https://ardupilot.org/dev/docs/porting.html).

## Aprire il progetto

Apri [Schedona gigante.kicad_pro](Schedona%20gigante/Schedona%20gigante.kicad_pro) dal gestore progetti di **KiCad 10**: i file presenti sono stati salvati con la versione 10.0.

Dal gestore puoi accedere allo schema elettrico e all’editor PCB. Mantieni insieme i file nella cartella `Schedona gigante/`, senza cambiarne i nomi.

## Struttura della cartella

```text
.
├── README.md
├── librerie/
│   ├── Libreria gigante.kicad_sym   # Libreria di simboli personalizzata
│   └── Libreria gigante.bak         # Backup della libreria
├── Schedona gigante/
│   ├── Schedona gigante.kicad_pro   # Progetto e impostazioni
│   ├── Schedona gigante.kicad_sch   # Schema elettrico
│   ├── Schedona gigante.kicad_pcb   # Circuito stampato
│   ├── Schedona gigante.kicad_prl   # Impostazioni locali
│   └── .history/                   # Cronologia locale KiCad
└── docs/
    ├── README.md                   # Indice dei documenti
    ├── STM32H743 datasheet.pdf
    ├── X7+CORE_RC03.pdf
    ├── X7_BASE_RC03.pdf
    └── X7_IMU_RC03.pdf
```

## Elementi da eliminare o semplificare

Le seguenti sono decisioni progettuali da applicare durante la ricostruzione; non descrivono modifiche già effettuate allo schema.

### Eliminare la separazione fra le schede

- **Connettore BASE–CORE a 80 pin:** eliminare entrambe le metà, `CON3` sulla BASE (`DF17(3.0H)-80DS-0.5V(57)`) e `CON1` sul CORE (`DF17(1.0H)-80DP-0.5V(57)`). Collegare direttamente sul PCB le alimentazioni e i segnali necessari.
- **Connettore CORE–IMU a 30 pin:** eliminare la coppia `DF18B-30DS-0.4V` / `DF18D-30DP-0.4V(51)`, integrando i sensori scelti e i relativi bus, chip select e interrupt sulla scheda unica.
- **Interconnessioni e ingombri fra moduli:** eliminare cablaggi fra schede, aree di accoppiamento e vincoli di impilamento. Distanziali, fissaggi e supporti destinati ai tre moduli andranno rimossi se presenti nell’assemblaggio, mantenendo i fissaggi necessari alla scheda unica.
- **Componenti usati soltanto per attraversare le interfacce interne:** riesaminare resistenze in serie, pull-up, buffer e protezioni associati ai collegamenti fra moduli. Rimuovere soltanto quelli divenuti superflui; conservare terminazioni, polarizzazioni e protezioni richieste dai circuiti o dalle porte esterne.
- **Distribuzione dell’alimentazione fra moduli:** eliminare i passaggi attraverso i connettori e accorpare eventuali stadi duplicati solo dopo aver verificato tensioni, correnti, rumore e sequenze di accensione. Conservare disaccoppiamento locale e alimentazione pulita dei sensori.

### Valutare le semplificazioni funzionali

Queste rimozioni sono candidate per ridurre area e complessità, e richiedono una scelta esplicita dei requisiti prima di modificare lo schema.

| Blocco nei PDF | Semplificazione da valutare | Condizione da rispettare |
| --- | --- | --- |
| Sensori e canali ridondanti: ADIS16470, BMI088, ICM-20689, MS5611 e relativi bus | Ridurre il numero di sensori e rimuovere i circuiti dedicati a quelli esclusi | Mantenere una IMU supportata e le misure richieste dal sistema; verificare driver, orientamento e prestazioni. Un barometro interno non sostituisce il sensore di pressione per la profondità. |
| Magnetometro RM3100 e circuito associato | Scegliere se mantenerlo a bordo oppure usare un sensore esterno | Definire le esigenze di stima della direzione e verificare le interferenze dei propulsori e dell’alimentazione. |
| Riscaldatore IMU da 1 W, MOSFET AO3400A e rete di comando `HEATER_EN` / `IMU_GPIO_HEATER` | Eliminare riscaldatore e circuito di pilotaggio se la stabilizzazione termica non è richiesta | Validare deriva dei sensori e funzionamento nell’intervallo di temperatura previsto; aggiornare la configurazione firmware. |
| USB High Speed: PHY USB3300, quarzo da 24 MHz e bus ULPI | Eliminare il ramo HS se è sufficiente la USB Full Speed del microcontrollore | Conservare una porta USB funzionante, protezioni e accesso al caricamento firmware. |
| Bridge e adattatori di livello: TXS0108E, SN74LVC8T245 | Ridurre i canali o eliminare gli stadi relativi a porte rimosse o a segnali interni con livelli compatibili | Verificare i livelli elettrici di ogni periferica esterna e degli ESC prima di collegarli direttamente. |
| Porte GPS, RC/PPM, UART, I²C, SPI e CAN non utilizzate | Eliminare connettori e circuiti dedicati alle porte escluse | Conservare le interfacce necessarie a telemetria, companion computer, sensore di profondità e periferiche del ROV. |
| Ingressi di alimentazione ridondanti e uscite di alimentazione ausiliarie | Ridurre ingressi, selettori e interruttori alle sole linee richieste | Verificare protezioni, assorbimenti, alimentazione via USB e assenza di ritorni di corrente. |
| LED RGB, buzzer, pulsanti e connettori di servizio duplicati | Ridurre gli indicatori e sostituire i connettori di debug ingombranti con punti di test | Conservare diagnostica essenziale, SWD, reset, accesso al boot e funzioni di sicurezza previste. |

La compattezza deve conservare le funzioni richieste dal ROV: uscite per tutti gli ESC e gli attuatori previsti, sensori scelti, collegamento MAVLink, misura della batteria e interfaccia di profondità. La configurazione di riferimento è descritta nella [documentazione hardware ArduSub](https://ardupilot.org/sub/docs/sub-hardware.html).

## Stato attuale

- Lo schema elettrico contiene una sezione intitolata **Power** e simboli del microcontrollore STM32H743.
- Il file PCB contiene soltanto l’intestazione: il layout della scheda è ancora da realizzare.
- L’integrazione dei tre moduli e la compatibilità ArduSub devono ancora essere completate e collaudate.

## Attività da completare

- [ ] Definire dimensioni massime, fissaggi, vincoli del contenitore, alimentazione e intervallo di temperatura.
- [ ] Definire numero di propulsori, uscite ESC, attuatori e periferiche del ROV, con porte e connettori necessari.
- [ ] Analizzare tutti i fogli dei tre PDF e compilare una matrice dei blocchi da mantenere, integrare o eliminare, indicando componenti e segnali coinvolti.
- [ ] Scegliere il target ArduSub e la versione firmware di riferimento; verificare supporto del microcontrollore e driver dei sensori prima di fissare la distinta componenti.
- [ ] Definire l’architettura su un unico PCB e approvare le rimozioni candidate riportate sopra.
- [ ] Ricostruire e verificare l’alimentazione: ingressi, regolatori, protezioni, misure di tensione e corrente, alimentazione sensori e periferiche.
- [ ] Completare il circuito STM32H743 con clock, disaccoppiamento, reset, boot, SWD e USB.
- [ ] Integrare i sensori scelti, definendo bus, indirizzi, chip select, interrupt e orientamento; predisporre l’interfaccia per il sensore di profondità.
- [ ] Assegnare i pin per PWM, UART, I²C, SPI, CAN e ADC; verificare timer, DMA, livelli elettrici e assenza di conflitti.
- [ ] Eliminare le due coppie di connettori fra moduli, collegare direttamente i segnali necessari e rimuovere i circuiti esclusi dalla matrice dei blocchi.
- [ ] Completare lo schema, associare i footprint, preparare la distinta componenti e risolvere gli errori ERC, motivando le eventuali eccezioni.
- [ ] Realizzare il layout compatto rispettando percorsi di ritorno, distribuzione dell’alimentazione, integrità dei segnali, posizione dei sensori e accessibilità dei connettori.
- [ ] Completare DRC e revisione di schema e PCB; generare Gerber, file di foratura e dati di assemblaggio.
- [ ] Preparare o adattare `hwdef.dat` e `hwdef-bl.dat` coerenti con il nuovo hardware; compilare bootloader e firmware ArduSub.
- [ ] Assemblare il prototipo e verificare alimentazioni, assorbimento, temperature, programmazione SWD, boot e collegamento USB.
- [ ] Collaudare riconoscimento e calibrazione dei sensori, orientamento IMU, profondità, batteria, telemetria MAVLink, logging previsto e tutte le uscite ESC.
- [ ] Verificare armamento, arresto dei propulsori e failsafe per perdita del collegamento e batteria; iniziare con prove al banco senza eliche.
- [ ] Eseguire prove controllate sul ROV e registrare risultati, problemi e modifiche necessarie.
- [ ] Finalizzare pinout, distinta componenti, istruzioni di assemblaggio e configurazione ArduSub; considerare raggiunto l’obiettivo solo dopo il collaudo della scheda unica.
