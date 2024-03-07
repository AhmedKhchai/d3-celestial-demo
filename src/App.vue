<template>
  <div id="app">
    <div id="celestial-map" />
    <label>Location: </label>
    <input v-model.number="lng" label="lng" type="number" />
    <input v-model.number="lat" label="lat" type="number" />
    Hours:
    <input v-model.number="hours_offset" label="+ hours" type="number" /> Date:
    <input v-model="dates" label="date" type="date" />
    <button @click="update_location">go</button>
    <button @click="now">now</button>
    <div>{{ [lat, lng] }}</div>
    <div>{{ getOffset(timezone) }}</div>
  </div>
</template>

<script>
import celestial from 'd3-celestial';

const Celestial = celestial.Celestial();
export default {
  name: 'App',
  data() {
    return {
      lat: 35,
      lng: -105,
      hours_offset: 0,
      timezone: 'America/Denver',
      FONT: 'Raleway',
      dates: null,
    };
  },
  mounted() {
    const config = {
      width: 800,
      projection: 'stereographic', // Map projection used: airy, aitoff, armadillo, august, azimuthalEqualArea, azimuthalEquidistant, baker, berghaus, boggs, bonne, bromley, collignon, craig, craster, cylindricalEqualArea, cylindricalStereographic, eckert1, eckert2, eckert3, eckert4, eckert5, eckert6, eisenlohr, equirectangular, fahey, foucaut, ginzburg4, ginzburg5, ginzburg6, ginzburg8, ginzburg9, gringorten, hammer, hatano, healpix, hill, homolosine, kavrayskiy7, lagrange, larrivee, laskowski, loximuthal, mercator, miller, mollweide, mtFlatPolarParabolic, mtFlatPolarQuartic, mtFlatPolarSinusoidal, naturalEarth, nellHammer, orthographic, patterson, polyconic, rectangularPolyconic, robinson, sinusoidal, stereographic, times, twoPointEquidistant, vanDerGrinten, vanDerGrinten2, vanDerGrinten3, vanDerGrinten4, wagner4, wagner6, wagner7, wiechel, winkel3
      // transform: 'equatorial', // Coordinate transformation: equatorial (default), ecliptic, galactic, supergalactic
      controls: false, // zoom controls

      // geopos: [this.lat, this.lng],
      orientationfixed: true,
      follow: 'zenith',
      disableAnimations: true,
      stars: {
        colors: false,
        size: 4,
        limit: 6,
        exponent: -0.26,
        designation: false,
        style: { fill: '#000000', opacity: 1 },
        propername: false,
        propernameType: 'name',
        propernameStyle: {
          fill: '#ddddbb',
          font: `8px ${this.FONT}`,
          align: 'right',
          baseline: 'center',
        },
        propernameLimit: 2.0,
      },
      dsos: {
        show: false,
        size: 6,
        designation: false,
        propername: false,
        data: 'dsos.6.json',
      },
      constellations: {
        names: false, // Show constellation names
        namesType: 'iau', // Type of name Latin (iau, default), 3 letter designation (desig) or other language (see list below)
        nameStyle: {
          fill: '#cccc99',
          align: 'center',
          baseline: 'middle',
          font: [
            '14px Helvetica, Arial, sans-serif', // Style for constellations
            '12px Helvetica, Arial, sans-serif', // Different fonts for diff.
            '11px Helvetica, Arial, sans-serif',
          ],
        }, // ranked constellations
        lines: true, // Show constellation lines, style below
        lineStyle: { stroke: '#cccccc', width: 1, opacity: 0.6 },
        bounds: false, // Show constellation boundaries, style below
        boundStyle: {
          stroke: '#cccc00',
          width: 0.5,
          opacity: 0.8,
          dash: [2, 4],
        },
      },
      planets: {
        show: false,
        which: [
          'sol',
          'mer',
          'ven',
          'ter',
          'lun',
          'mar',
          'jup',
          'sat',
          'ura',
          'nep',
        ],
        // Font styles for planetary symbols
        symbolType: 'disk',
        symbols: {
          // Character and color for each symbol in 'which', simple circle \u25cf
          sol: { symbol: '\u2609', letter: 'Su', fill: '#ffff00', size: '24' },
          mer: { symbol: '\u263F', letter: 'Me', fill: '#cccccc' },
          ven: { symbol: '\u2640', letter: 'V', fill: '#eeeecc' },
          ter: { symbol: '\u2295', letter: 'T', fill: '#00ccff' },
          lun: { symbol: '\u25CF', letter: 'L', fill: '#ffffff', size: '24' }, // overridden by generated crecent, except letter & size
          mar: { symbol: '\u2642', letter: 'Ma', fill: '#ff6600' },
          cer: { symbol: '\u26B3', letter: 'C', fill: '#cccccc' },
          ves: { symbol: '\u26B6', letter: 'Ma', fill: '#cccccc' },
          jup: { symbol: '\u2643', letter: 'J', fill: '#ffaa33' },
          sat: { symbol: '\u2644', letter: 'Sa', fill: '#ffdd66' },
          ura: { symbol: '\u2645', letter: 'U', fill: '#66ccff' },
          nep: { symbol: '\u2646', letter: 'N', fill: '#6666ff' },
          plu: { symbol: '\u2647', letter: 'P', fill: '#aaaaaa' },
          eri: { symbol: '\u26AA', letter: 'E', fill: '#eeeeee' },
        },
        names: true,
        nameStyle: {
          fill: '#cccccc',
          font: "17px 'Lucida Sans Unicode', 'DejaVu Sans'",
          align: 'right',
          baseline: 'top',
        },
        namesType: 'desig',
      },
      mw: {
        show: false,
      },
      lines: {
        // Display & styles for graticule & some planes
        graticule: {
          show: true,
          stroke: '#cccccc',
          width: 0.6,
          opacity: 0.8,
          // grid values: "outline", "center", or [lat,...] specific position
          lon: {
            pos: [''],
            fill: '#eee',
            font: '10px Helvetica, Arial, sans-serif',
          },
          // grid values: "outline", "center", or [lon,...] specific position
          lat: {
            pos: [''],
            fill: '#eee',
            font: '10px Helvetica, Arial, sans-serif',
          },
        },
        equatorial: { show: true, stroke: '#aaaaaa', width: 1.3, opacity: 0.7 },
        ecliptic: { show: false, stroke: '#66cc66', width: 1.3, opacity: 0.7 },
        galactic: { show: false, stroke: '#cc6666', width: 1.3, opacity: 0.7 },
        supergalactic: {
          show: false,
          stroke: '#cc66cc',
          width: 1.3,
          opacity: 0.7,
        },
      },
      background: {
        // Background style
        fill: '#ffffff', // Area fill
        opacity: 1,
        stroke: '#00000', // Outline
        width: 1.5,
      },
    };
    this.$nextTick(() => {
      console.log(config);
      Celestial.display(config);
      Celestial.location([this.lat, this.lng]);
      // this.now();
    });
  },
  methods: {
    /* @return A timezone offset in minutes */
    getOffset(timeZone = 'UTC', date = new Date()) {
      const utcDate = new Date(
        date.toLocaleString('en-US', { timeZone: 'UTC' })
      );
      const tzDate = new Date(date.toLocaleString('en-US', { timeZone }));
      return (tzDate.getTime() - utcDate.getTime()) / 6e4;
    },
    now() {
      Celestial.date(new Date()); //, this.getOffset(this.timezone));
    },
    update_location() {
      if (!this.date) {
        const time = new Date();
        time.setHours(time.getHours() + this.hours_offset); // include offset
        Celestial.date(time);
      }
      let date = new Date(this.dates);
      date.setHours(date.getHours() + this.hours_offset);
      Celestial.date(date);
      Celestial.location([this.lat, this.lng]);
    },
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>