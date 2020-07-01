# SerialTalks

<aside class="notice">
La librairie SerialTalks assure la communication entre ordinateur(PC ou Raspberry) et arduino. On trouve dans cette librairie deux versions differentes pour chaque terminal.
</aside>

## Préambule

Cette librairie de communication se base sur des requêtes et des retours. Chaque packet à une forme standardisé. (Add image)

### Query Parameters

| ordre | Nom         | Description                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Master Code | Le Master code permet de faire la différence entre un packet esclave et maître. Un packet maître corresponds à une demande alors que le packet esclave est une réponse suite à une demande. Le Master code permet de faire la difference entre un packet esclave et maitre. Un packet maitre correspond a une demande alors que le packet esclave est un retour suite a une demande. Il vaut soit b'R' (maitre) soit b'A' (esclave). |
