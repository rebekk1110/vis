<script>
  import { scaleLinear, scaleBand,scaleSequential } from "d3-scale";
  import { interpolateBlues, interpolateMagma} from "d3-scale-chromatic";
  import { autoType, csvParse } from "d3-dsv";
  import { Graphic,YAxis,XAxis,Label,Section, PolygonLayer, fitScales,RectangleLayer, Rectangle,Point } from "@snlab/florence";
  import DataContainer from "@snlab/florence-datacontainer";
  import {provinces} from '../data/prov.js'


  //Hente data csv
  let done = false;
  let streetDataRaw;
  let streetData;

  let streetCountAll
  let streetCountFemale

  let myColorScale
  let percentFemale=[]

  let selectProv='**All provinces**'

  const prov = new DataContainer(provinces)
  console.log(prov)
  const myGeoScale = fitScales(prov.domain('$geometry'))


  

  async function loadCSV(url) {
    const response = await fetch(url);
    const data = await response.text();
    const dataParsed = csvParse(data);
    streetDataRaw = new DataContainer(dataParsed);
    streetData = streetDataRaw.rename({ lau_name: 'region' ,instance_of_label:'human_type'})

    //Create new concetrated dataset
    streetCountAll = streetData
    .filter((obj)=>obj.person==1)
    .groupBy('region')
    .summarise({ numberPersons: { gender: 'count' } })
    streetCountFemale=streetData  
    .filter((obj)=>obj.gender=='female')
    .groupBy('region')
    .summarise({ numberFemale: { gender: 'count' } })

    streetCountFemale.setKey('region')
    streetCountAll.addColumn('numberFemale', streetCountAll.map('region', region => streetCountFemale.row({key:region})['numberFemale']))
    percentFemale=calcPercentArray(streetCountAll.column('numberFemale'),streetCountAll.column('numberPersons'))
    streetCountAll.addColumn('percentFemale',percentFemale)
    console.log(streetCountAll)
    
    streetCountAll.setKey('region')
    prov.addColumn('percentFemale', prov.map('prov_name', region => streetCountAll.row({key: region})['percentFemale']))

    console.log("--")
  
    myColorScale=scaleSequential().domain(prov.domain('percentFemale')).interpolator(interpolateBlues)
    



    done = true;
  }
    

  function myColorScalet(input){
    if(input==undefined){
      return('rgb(238,238,228)')
    }
    else{
       console.log(myColorScale(input))
      return myColorScale(input)
    }
  }

   $: {
    if (selectProv != '**All provinces**') {
      console.log(selectProv)

    }
  }
  
  loadCSV("data/allstreet3.csv");

  function calcPercentArray(part,total){
    let percent=[]
    for (let i=0; i<part.length; i++){
    percent.push(parseFloat(((parseFloat(part[i])/parseFloat(total[i]))*100).toFixed(2)))
    }
  return percent
  }



</script>



  <div class="main-chart">
    <h1>Mitt kart</h1>
<div class="graph">
    {#if done}
      <div>
    <label for="hood-select">Choose a province:</label>
    <select bind:value={selectProv} id="hood-select" >
      {#each streetCountAll.column('region') as region}
        <option value={region}>
          {region}
        </option>
        {/each}
	    </select>
      </div>
     <Graphic width={750} height={800} >
      <Section
        x1={0}
        x2={0.14}
        y1={0.1}
        y2={0.6}
        scaleX={[0, 10]}   
        scaleY={[0, 9]}
        padding={10}  
        flipY      
        >

        {console.log("hei")}
        {#each streetCountAll.column('percentFemale').sort() as item,i}
 
          <Rectangle
          x1={0}
          x2={3}
          y1={i}
          y2={1+i}
          stroke={'black'}
          fill={myColorScale(item)}
          />
          <Label
          x={7}
          y={0.5+i}
          text={item+"%"}
          />
        {/each}

      </Section>
      <Section
        x1={0.15}
        x2={1}
        y1={0}
        y2={0.8}
        flipY
    
        {...myGeoScale}
      >
        <PolygonLayer 
          geometry={prov.column('$geometry')}
          stroke={prov.map('prov_name',(e)=>e==selectProv ? "red":"gray")}
          strokeWidth={prov.map('prov_name',(e)=>e==selectProv ? 3:"gray")}
          fill={prov.map('percentFemale',myColorScalet)}
        />
      </Section>  
      <Section
        x1={0.15}
        x2={0.7}
        y1={0.8}
        y2={0.88}
        scaleX={scaleLinear().domain(streetCountAll.domain('numberFemale'))}
        scaleY={[0,10]}
        >
        {console.log(streetCountFemale.row({key:'Torino'})['numberFemale'])}
        <Rectangle
          x1={3}
          x2={streetCountFemale.row({key:selectProv})['numberFemale']}
          y1={5}
          y2={9}
        />
        <YAxis  tickCount = {0}/>
        <XAxis title="Number of street called after a woman" tickCount = {10}/>
    
      </Section>

    </Graphic>
   
    {/if}
    </div>

  <p>
    In  our project we will visualize the amount of streets named after women in different countries in europ.
  </p>
  <p>
    In my assigment I have coosen a smaller domain and I choosed cities in Italy.
    I choose a choropleth map since it can display both differences and geographic localtion of the data. 
  </p>
  <p>
    One of my design choices was to choose percent and not the count of female street. The reason behind this was that I wanted to focus on the differences between the cities in Italy, rather than the amout.
  </p>
</div>
<style>

 /* header styling */
  h1,
  h2,
  h3 {
    font-family:Arial, Helvetica, sans-serif;
    font-weight: 200;
    text-align: center;
    color: rgb(60, 60, 60);
  }

  h1 {
    font-size: 20px;
  }

 
  .graph {
    width: 750px;
    background-color: rgb(223, 216, 212);
    padding: 10px;
  }



</style>
