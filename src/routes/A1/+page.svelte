<script lang="ts">
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import type { TMovie } from "../../types";
  import Bar from "$lib/Bar.svelte";
  import Q1MultiLineChart from "$lib/Q1MultiLineChart.svelte";
  import Q2Heatmap from "$lib/Q2Heatmap.svelte";

  // Reactive variable for storing the data
  let movies: TMovie[] = [];

  // Function to load the CSV
  async function loadCsv() {
    try {
      const csvUrl = "./summer_movies.csv";
      movies = await d3.csv(csvUrl, (row) => {
        // TIP: in row, all values are strings, so we need to use a row conversion function here to format them
        return {
          // ...row, // spread syntax to copy all properties from row
          // num_votes: Number(row.num_votes),
          // year: new Date(row.year),
          // please also format the values for other non-string attributes. You can check the attributes in the CSV file
          tconst: row.tconst,
          title_type: row.title_type,
          primary_title: row.primary_title,
          original_title: row.original_title,
          // Convert the year (assumed to be a number) into a Date (using January 1st of that year)
          year: new Date(Number(row.year), 0, 1),
          runtime_minutes: Number(row.runtime_minutes),
          // Split the genres string into an array (assumes comma separation)
          genres: row.genres.split(","),
          average_rating: Number(row.average_rating),
          num_votes: Number(row.num_votes)
        } as TMovie;
      });

      console.log("Loaded CSV Data:", movies);
    } catch (error) {
      console.error("Error loading CSV:", error);
    }
  }
  // Call the loader when the component mounts
  onMount(loadCsv);
</script>

<h1>Summer Movies</h1>

<p>Here are {movies.length == 0 ? "..." : movies.length + " "} movies</p>
<Bar {movies} />
<Q1MultiLineChart {movies} />
<Q2Heatmap {movies} />