Les “labels” **marchent**, mais dans ton code ils sont **bridés** : ils ne s’affichent que si **(mode === "GRAPH") OU (zoom >= 0.95)**. Donc en ZONES + zoom un peu bas → tu crois que le switch ne fait rien. C’est exactement ici dans ton fichier  :

```js
const canLabel = showLabels && (mode==="GRAPH" || view.z>=0.95);
```

### Fix simple (labels = ON ⇒ toujours visibles)

Cherche ce bloc dans `draw()` et remplace-le comme suit :

```js
// AVANT
const canLabel = showLabels && (mode==="GRAPH" || view.z>=0.95);

// APRÈS (labels = toujours visibles si activés)
const canLabel = showLabels;
```

### Bonus (si tu veux garder un seuil, mais “qui a du sens”)

Tu peux mettre un seuil plus permissif et indépendant du mode :

```js
const canLabel = showLabels && view.z >= 0.70;
```

### Option “propre” (éviter le surchargement en ZONES)

Garde l’affichage mais **limite** les labels quand y’en a trop :

Remplace ton bloc label par ceci :

```js
if (canLabel){
  const nm = ui.reveal.checked
    ? (e.display.real_name||e.display.masked_name||e.id)
    : (e.display.masked_name||e.id);

  // limite auto: en ZONES, on n'affiche pas les labels si on est trop zoom-out
  if(mode==="ZONES" && view.z < 0.85) {
    // rien
  } else {
    ctx.fillStyle="rgba(234,241,255,.80)";
    ctx.font=`${Math.max(10,11*view.z)}px ${getComputedStyle(document.documentElement).getPropertyValue('--mono')||'monospace'}`;
    ctx.fillText(nm, s.x+10*view.z, s.y-8*view.z);
  }
}
```

Si tu appliques le **fix simple**, ton switch “Labels” redevient “bête et méchant” : ON = visible, OFF = invisible, sans surprise.
