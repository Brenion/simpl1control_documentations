

```js
function calculIndex(heure, minute) {
  // Étape 1 : conversion de la minute en "valeur base 16 étendue"
  const minuteVal = minute + 6 * Math.floor(minute / 10);

  // Étape 2 : base de départ selon la plage horaire
  let baseOffset;
  if (heure < 10) {
    baseOffset = heure * 256;
  } else if (heure < 20) {
    baseOffset = 4096 + (heure - 10) * 256;
  } else {
    baseOffset = 8192 + (heure - 20) * 256;
  }

  // Étape 3 : index final
  return baseOffset + minuteVal;
}
```
