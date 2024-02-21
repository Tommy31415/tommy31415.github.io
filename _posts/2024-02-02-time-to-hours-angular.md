---
layout: post
title: Kendo Circural and time in seconds that cross midnight
date: 2024-02-02
categories: typescript chatgpt
---


# Kendo Circural and time in seconds that cross midnight

Input values are negative and positive. Negative values happened before midnight and positive after.

## Convert seconds into hours:

   ```typescript
    function convertSecondsToHHMM(seconds: number): string {
    const absSeconds = Math.abs(seconds);
    const hours = Math.floor(absSeconds / 3600);
    const minutes = Math.floor((absSeconds - (hours * 3600)) / 60);

    // Returns time in hh:mm format, adds a '-' prefix if the original value was negative
    return (seconds < 0 ? "-" : "") + String(hours).padStart(2, '0') + ":" + String(minutes).padStart(2, '0');
    }
   ```

## Kendo Cicrular

```html
<kendo-circulargauge [value]="executionTime" [scale]="gaugeScale" (valueChange)="onValueChange($event)">
    <kendo-circulargauge-scale
        [majorTicks]="{ visible: true }"
        [minorTicks]="{ visible: true }"
        [labels]="{ position: 'inside' }">
    </kendo-circulargauge-scale>
</kendo-circulargauge>
```

Consider:

```html
<kendo-circulargauge [value]="executionTime" [scale]="gaugeScale">
    <kendo-circulargauge-pointer [color]="executionTime < 0 ? '#3498db' : '#e74c3c'"></kendo-circulargauge-pointer>
</kendo-circulargauge>
```

## TS

```javascript
gaugeScale = {
  majorUnit: 1, // Defines the interval between major ticks
  min: -12, // Assuming the minimum value represents jobs finished 12 hours early
  max: 12, // Assuming the maximum value represents jobs that took up to 12 hours longer
  labels: {
    format: '{0}h',
    position: 'inside', // Positions labels inside the gauge for a cleaner look
  },
  minorUnit: 0.5, // Defines the interval between minor ticks for finer granularity
};
```

