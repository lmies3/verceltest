<script>
  import { Grid, Row, Column } from "carbon-components-svelte";
  import { Tabs, Tab, TabContent } from "carbon-components-svelte";
</script>

<svelte:head>
	<title></title>
</svelte:head>

<Grid narrow>
  <Row>
    <Column>
      <h2>About</h2>
      <hr>
    </Column>
  </Row>
  <Row>
    <Column>
      <Tabs>
        <Tab label="Usage" />
        <Tab label="Background" />
        <svelte:fragment slot="content">
          <TabContent>Usage Instrucstions</TabContent>
          <TabContent>Background Literature</TabContent>
        </svelte:fragment>
      </Tabs>
    </Column>
  </Row>
</Grid>