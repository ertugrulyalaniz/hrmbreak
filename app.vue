<script setup>
import { read, utils, writeFile } from 'xlsx';
import '@picocss/pico';
const dropLink = async (e) => {
  e.stopPropagation();
  e.preventDefault();
  const f = e.dataTransfer.files[0];
  const data = await f.arrayBuffer();

  const workbook = read(data, { type: 'binary', cellDates: true });

  var jsa = utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], {
    header: 1,
    cellDates: true,
  });
  dataParser(jsa);
};

const breakTimeCalc = (data) => {
  let totalBreakTime = 0;

  for (let i = 0; i < data.length - 1; i++) {
    if (data[i].type === 'CKS' && data[i + 1].type === 'GRS') {
      let breakTime = data[i + 1].time - data[i].time;
      totalBreakTime += breakTime;
    }
  }

  totalBreakTime = totalBreakTime / 1000 / 60; // to convert from milliseconds to minutes
  console.log('Total break time in minutes: ', totalBreakTime);
  return totalBreakTime.toFixed(0);
};

let final = [];
const result = ref();

const parseDateString = (dateString) => {
  var parts = dateString.split(/[/: ]/); // Split the string into an array of parts

  // Extract the date and time components
  var day = parseInt(parts[0], 10);
  var month = parseInt(parts[1], 10) - 1; // Months in JavaScript are zero-based (0-11)
  var year = parseInt(parts[2], 10);
  var hour = (parseInt(parts[3], 10) % 12) + (parts[6] === 'pm' ? 12 : 0); // Adjust for AM/PM
  var minute = parseInt(parts[4], 10);
  var second = parseInt(parts[5], 10);

  // Create a new Date object
  var date = new Date(year, month, day, hour, minute, second);

  return date;
};

const dataParser = (jsson) => {
  // 9:15 den sonrasini hesapla
  // 18:00 den sonrasini hesapla
  for (let index = 1; index < jsson.length; index++) {
    final.push({
      id: jsson[index][1],
      name: jsson[index][2],
      department: jsson[index][3],
      type: jsson[index][5].includes('CKS') ? 'CKS' : 'GRS',
      time: parseDateString(jsson[index][6]),
      cardNo: jsson[index][7],
    });
  }
  result.value = final.reduce(function (r, a) {
    r[a.id] = r[a.id] || [];
    r[a.id].push(a);
    return r;
  }, Object.create(null));

  console.log(result.value);
};
</script>

<template>
  <main class="container">
    <article
      @drop.prevent="dropLink($event)"
      @dragenter.prevent
      @dragover.prevent
    >
      Drop Excel heree
    </article>

    <details v-for="res in result">
      <summary
        :style="
          breakTimeCalc(res) > 105
            ? 'background:#ffdada; padding:16px'
            : 'padding:16px;'
        "
      >
        {{ res[0].name }} - {{ res[0].id }} - {{ res[0].department }}
        <br />
        <span>Toplam mola: {{ breakTimeCalc(res) }} dk</span>
      </summary>
      <table>
        <thead>
          <tr>
            <th scope="col">Action</th>
            <th scope="col">Time</th>
          </tr>
        </thead>
        <tbody v-for="r in res">
          <tr>
            <td>{{ r.type }}</td>
            <td>{{ new Date(r.time) }}</td>
          </tr>
        </tbody>
        <tfoot>
          <tr>
            <td scope="col">Total</td>
            <td scope="col">{{ breakTimeCalc(res) }}</td>
          </tr>
        </tfoot>
      </table>
    </details>
  </main>
</template>
<style>
html,
body {
  padding: 0;
  margin: 0;
  display: block;
  flex-direction: row;
  min-width: 100%;
  font-family: sans-serif;
  background: var(--dark-black);
  padding-top: 50px;
}
.w-full {
  display: flex;
  width: 100%;
}
</style>
