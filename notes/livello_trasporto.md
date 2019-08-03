# Livello di Trasporto - TCP e UDP
I protocolli di livello di trasporto devono fornire la comunicazione logica tra
processi applicativi di host differenti, detta anche comunicazione end-to-end.
Si occupa principalmente della frammentazione e ricomposizione dei segmenti
spediti e ricevuti, anche in ordine diverso.
Il livello trasporto non ha visibilita' della sulla rete, perche' deve
mascherarla al livello applicativo; fornisce, quindi primitive di scrittura e
lettura.

### mux/demux
A Livello Trasporto la destinazione non e' un host ma un processo in esecuzione
su un processo in esecuzione su un host.
L'interfaccia tra processi applicativi e' rappresentata  da una porta:
- numero intero su 16 bit
- associazione tra porte e processi
- esistono le well-know port (o statiche): tra 0-1024 che sono standard e 
  pubblici
- da 1025-2^16 sono porte libere (o dinamiche)
Ovviamente la porta del client, nel caso di stabilire una connessione, puo'
essere una porta scelta a caso dal sistema operativo.

Un flusso dati e' identificato da 5 parametri:
1. Porta Sorgente                   [TCP]
2. Porta Destinazione               [TCP]
3. Indirizzo di Sorgente            [IP]
4. Indirizzo di Destinazione        [IP]
5. Protocollo di livello trasporto  [IP]

## TCP
Il protocollo TCP fornisce servizi orientati alla connessione, quindi affidabili
con le seguenti caratteristiche:
- ogni segmento inviato viene riscontrato singolarmente
- mantiene lo stato della connessione: instaurazione e rilascio 
- recupero degli errori (sui dati)
- consegna ordinata dei segmenti: essendo che non conosce la rete puo'
  succedere che i segmenti non arrivino nell'ordine di invio.
- controllo di flusso: controlli per evitare che il DESTINATARIO non riesca
  a immagazzinare tutti i dati che sto trasmettendo
- controllo di congestione: controlli per evitare che la RETE non riesca a
  consegnare tutti i dati che sto trasmettendo

## UPD
Protocollo leggero e veloce, in quanto non garantisce molte funzionalita' che
TCP invece fornisce. UDP non e' orientato alla connessione quindi non e'
affidabile. Ha solo la caratteristica che
- ogni segmento inviato non viene riscontrato: in questo modo non c'e'
  sistema di riscontro dell'avvenuta ricezzione per la sorgente 

In questo modo e' possibile una duplicazione dei pacchetti UDP da parte del 
ricevente, ma e' un caso gestito puramente dal livello applicazione.

Il segmento UDP e' composto:
- Porta sorgente e destinazione [16bit]
- lunghezza del pacchetto, compreso header [16bit]
- checksum per l'header [16bit]

Non e' possibile usare direttamente IP sottostante, perche' IP non e' 
end-to-end in quanto non ha porta sorgente-destinazione.

### Checksum UDP
Obbiettivo: rilevare gli errori del segmento trasmesso.
Trasmettitore:
- calcola la checksum del segmento
- lo allega nel campo specificato
- invia il segmento

Ricevitore:
- riceve il segmento
- calcola la checksum sul segmento senza il campo checksum
- lo confronta con il campo checksum del segmento: se sono uguali il
      segmento viene passato al livello applicativo, altromenti viene buttato.

Una volta identificato l'errore ( UDP butta via tutto ) TCP usa una serie
di protocolli che prendono il nome di  ARQ - Automatic Retransmission reQuest;
ovvero la decisione di ritrasmettere un segmento viene presa in base ad una
finestra di trasmissione e dai riscontri ottenuti.
E' possibile anche correggere gli errori, tramite: bit di parita', codice a
ripetizione, ecc.


### Protocolli a Finestra Stop and Wait
I protocolli di Trasporto a Finestra Scorrevole di tipo Stop and Wait funzionano
nel seguente modo:

Trasmettitore
- invia un PDU e ne salva una copia
- attiva un timer ad un tempo standard
- si pone in attesa della ricezione dell ACK
- se scade il timer, ritrasmette la PDU, altrimenti, se nel frattempo riceve
      l'ACK, passa all PDU successiva.
    
Ricevitore
- controlla la correttezza della PDU
- controlla il numero di sequenza: in pratica controlla se si aspetta un
  numero di segmento che e' all'interno della sua finestra di ricezione
- se la PDU e' corretta, invia ACK e la consegna al livello superiore,
  altrimenti butto via la PDU

[immagine da slide, p22]

Il protocollo e poco efficiente perche' per ogni trasmissione devo attendere il
suo ACK. La soluzione e' la trasmissione in pipeline delle PDU, ovvero una
trasmissione in sequenza.

Si definisce un a Finestra di Trasmissione delle PDU, Wt come la quantita'
massima di PDU in sequenza che il trasmettitore e' autorizzato ad inviare in
rete senza averne ricevuto l'ACK.
NB: La Finestra di Trasmissione conta anche gli ACK come pacchetti presenti
sulla rete.
Anche il ricevitore ha una Finestra di Ricezione, Wr, in modo da ricevere tutto
quello che trasmettitore gli manda e garantire che ogni segmento inviato (anche
in disordine) venga ricevuto.
Questo comporta il problema di inizializzare Wr ad una dimensione appropriata,
in quanto se e' troppo piccola il ricevitore rischia di dover inviare un ACK per
segmento, quindi vanifica l'uso della Finestra Scorrevole, e neppure troppo
grande perche' comporta uno spreco di memoria RAM. Quindi Wr deve essere almeno
uguale a Wt per essere usata.
In questo modo si fa controllo di flusso: basta un campo nell'intestazione
degli ACK dove il ricevente  specifica la dimensione della finestra di
ricezione, in unita' di  PDU del protocollo in uso, in questo modo il
trasmettitore non inviera' piu di quello che il ricevente puo' gestire.

ESEMPIO:  
PDUt : PDU trasmettitore = 1kB  
Wr   : Finestra del ricevitore = 1k PDUs  
 --> Br : buffer di ricezzione = PDUt * Wr = 1MB  
 --> con 10^3 connessioni, la quantia' di memoria allocata = Br * 10^3 = 1GB


    
