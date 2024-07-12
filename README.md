# TARA ScaffoldVuer

Specific tool for creating acupuncture points and needles.

## How to use

### Creating Points

1. Turn on Quick Edit at the top right corner
2. Click on a desired location on the surface
3. A dialog will appear, enter the region and group name then press confirm to create a point.

### Creating Needles
1. Click on user created points.
2. A temporary lines normal to the nearest surface and a dialog will also appear, enter the region and group name then press confirm to create a line.

### Modifying Needles
1. Needles can be lengthened/shortened and move using the primitives control.
2. The control for the needles can be bought up after clicking on the corresponding item in the tree widget on the left.

### Displaying needles information
1. Needles information can be viewed by pressing on the Display Needles Informaion button on the top right.
2. Currently avaialable information include the name of objects in contact, distancce between the head and contact points, and coordinates of the contact point is currently 

### Import And Export Points and Needles
1. Points and Needles can be imported and exported using the Import / Export button on the top right. 

### To run in local development mode
```bash
npm run serve
```

## TARA ScaffoldVuer on NPM

Scaffoldvuer is available on npm and can be installed into your project with the following command:
```bash
npm i @abi-software/tara-scaffoldvuer
```

## How to use
Install the package in your vue app project with the following command "npm i @abi-software/scaffoldvuer".
Import the package in your script as followed:
```javascript
<script setup>
import { TaraScaffoldVuer } from '@abi-software/tara-scaffoldvuer';
import "@abi-software/tara-scaffoldvuer/dist/style.css";
</script>

<template>
  <div id="app">
    <TaraScaffoldVuer :url="url"/>
  </div>
</template>

<script>
export default {
  name: "app",
  data: function () {
    return {
      url: "Some URL"
    }
  }
}
</script>
```
