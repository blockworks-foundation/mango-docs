---
description: We are currently testing a WebSocket fills feed for all perpetual markets.
---

# Fills WebSocket Feed

This service is still being developed and may change.\
\
WebSocket data is streamed directly from the Mango validators, and in theory, should reduce the latency at which you receive fill event notifications.\
\
Additionally, the WebSocket stream will also revoke prior fill notifications in the event of a Solana fork.\
\
We currently have 3 WebSocket servers, in 3 different locations:

* Amsterdam: 89.233.107.242
* Chicago: 149.255.39.206
* Tokyo: 89.233.104.80

When you initially connect to either of these servers, you will be sent an initial data dump which contains a snapshot of past fills. The events data will need to be deserialised and parsed into EventQueue.

```json
{"market":"FTT-PERP","queue":"5pHAhyEphQRVvLqvYF7dziofR52yZWuq8DThQFJvJ7r5","events":[]}
{"market":"MNGO-PERP","queue":"7orixrhZpjvofZGWZyyLFxSEt2tfFiost5kHEzd7jdet","events":[]}
{"market":"SOL-PERP","queue":"31cKs646dt1YkA3zPyxZ7rUAkxTBz279w4XEobFXcAKP","events":[]}
{"market":"LUNA-PERP","queue":"HDJ43o9Dxxu6yWRWPEce44gtCHauRGLXJwwtvD7GwEBx","events":[]}
{"market":"BNB-PERP","queue":"GmX4qXMpXvs1DuUXNB4eqL1rfF8LeYEjkKgpFeYsm55n","events":[]}
{"market":"SRM-PERP","queue":"BXSPmdHWP6fMqsCsT6kG8UN9uugAJxdDkQWy87njUQnL","events":[]}
{"market":"ADA-PERP","queue":"G6Dsw9KnP4G38hePtedTH6gDfDQmPJGJw8zipBJvKc12","events":[]}
{"market":"ETH-PERP","queue":"9vDfKNPJkCvQv9bzR4JNTGciQC2RVHPVNMMHiVDgT1mw","events":[]}
{"market":"AVAX-PERP","queue":"5Grgo9kLu692SUcJ6S7jtbi1WkdwiyRWgThAfN1PcvbL","events":[]}
{"market":"BTC-PERP","queue":"7t5Me8RieYKsFpfLEV8jnpqcqswNpyWD95ZqgUXuLV8Z","events":[]}
{"market":"RAY-PERP","queue":"Css2MQhEvXMTKjp9REVZR9ZyUAYAZAPrnDvRoPxrQkeN","events":[]}
```

You will then receive events with a status of either "New" or "Revoke". The event data will need to be deserialised and parsed into FillEvent.

A "New" event is a single fill event for a particular market:

```json
{"market":"SOL-PERP","queue":"31cKs646dt1YkA3zPyxZ7rUAkxTBz279w4XEobFXcAKP","status":"New","event":""}
```

A "Revoke" event is a notification that a previous fill has been revoked due to a fork in the Solana blockchain and should be voided/ignored.

```json
{"market":"BTC-PERP","queue":"7t5Me8RieYKsFpfLEV8jnpqcqswNpyWD95ZqgUXuLV8Z","status":"Revoke","event":""}
```
