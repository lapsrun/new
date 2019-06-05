<script>
  // https://stackoverflow.com/a/7343013
  function round(value, precision) {
    var multiplier = Math.pow(10, precision || 0)
    return Math.round(value * multiplier) / multiplier
  }

  let title = ''
  let distance = 400
  let googleMapUrl = ''
  let diameter = 1
  let shoulder = 1
  let homeLanes = ''
  let lapLanes = ''
  let surfaceType = 'artificial'
  let surfaceColor = 'red'
  let steepleLocation = ''
  let features = ''

  let output = ''

  function handleSubmit() {


    toYaml({title, distance, googleMapUrl, diameter, shoulder, homeLanes, lapLanes, surfaceType, surfaceColor, steepleLocation, features}, (err, yaml) => {
      if(err) console.log(err)
      output = yaml
    })
  }


  function toYaml (opts={}, cb) {
    let {title, googleMapUrl, diameter = 2, distance, shoulder = 4, features} = opts

    googleMapUrl = googleMapUrl.replace(/\s/g, '')

    // acceptable values:
      // 34.0471033,-118.3759368
      // 34.0471033,    -118.3759368
      // https://www.google.com/maps/@34.0471033,-118.3759368,16z
      // https://www.google.com/maps/place/A+Food+Affair/@34.0517905,-118.3887014,16z/data=!4m5!3m4!1s0x80c2b9670a44a0d7:0x89cfb823c90e52fd!8m2!3d34.0517905!4d-118.384324

    // https://stackoverflow.com/questions/3518504/regular-expression-for-matching-latitude-longitude-coordinates
    let latLonPat = /[-+]?([1-8]?\d(\.\d+)?|90(\.0+)?),\s*[-+]?(180(\.0+)?|((1[0-7]\d)|([1-9]?\d))(\.\d+)?)/
    let latLon = googleMapUrl.match(latLonPat)[0]

    let [latitude, longitude] = latLon.split(',')

    let a = diameter / 2
    let c = shoulder - diameter
    let b = Math.sqrt(Math.pow(c, 2) - Math.pow(a, 2))
    let ab = (1 - a/b) * 100

    let shape = 1 - ((diameter / 2) / b)
    console.log('shape', shape)

    if (shape >= .05) {
      console.log('Incorrect Measurement Likely')
    } else if (shape >= -.10) {
      console.log('Single Radius')
    } else if (shape >= -.20) {
      console.log('Double Bend (1.2.3b/d)')
    } else if (shape >= -.40) {
      console.log('Double Bend (1.2.3c)')
    } else {
      console.log('Rectangle')
    }

    // let elevationSvc = new google.maps.ElevationService
    //
    // let elevOpts = {
    //   locations: [{
    //     lat: parseFloat(latitude),
    //     lng: parseFloat(longitude)
    //   }]
    // }
    //
    // elevationSvc.getElevationForLocations(elevOpts, (results, status) => {
    //   console.log(`elevation service status: ${status}`)
    //   let elevation = results[0].elevation
console.log(features)
      return cb(null, `---
  title: "${title}"
  date: ${(new Date).toISOString()}
  tags: [${features.map((f)=>{ return `"${f}"` }).join(", ")}]
  latitude: ${latitude}
  longitude: ${longitude}
  elevation_meters: -1
  distance_meters: ${distance}
  lap_lanes: ${opts.lapLanes}
  home_lanes: ${opts.homeLanes}
  surface_type: ${opts.surfaceType}
  colors:
    - ${opts.surfaceColor}
  turn_diameter_meters: ${round(diameter, 2)}
  turn_radius_b_meters: ${round(b, 2)}
  steeple_water_location: ${opts.steepleLocation}
  ---

  <!--more-->`)
    // })
  }

</script>

<form name="track" on:submit|preventDefault={handleSubmit}>
  <h2>Required</h2>
  <div>
    <label>Title:</label>
    <input bind:value={title} />
  </div>
  <div>
    <label>Distance (meters): </label>
    <input bind:value={distance} />
  </div>
  <div>
    <label>Google Maps URL</label>
    <input bind:value={googleMapUrl} />
  </div>
  <div>
    <label>Turn Diameter (meters)</label>
    <input bind:value={diameter} />
  </div>
  <div>
    <label>Shoulder to Head distance (meters)</label>
    <input bind:value={shoulder} />
  </div>

  <h2>Optional</h2>
  <div>
    <label># of Lanes (home straightaway)</label>
    <input bind:value={homeLanes} />
  </div>
  <div>
    <label># of Lanes (lap)</label>
    <input bind:value={lapLanes} />
  </div>
  <div>
    <label>Surface Type</label>
    <input bind:value={surfaceType} />
  </div>
  <div>
    <label>Surface Color</label>
    <input bind:value={surfaceColor} />
  </div>
  <div>
    <label>Steeplechase Water Location</label>
    <select bind:value={steepleLocation}>
      <option value="n/a">Not Applicable</option>
      <option value="inside">Inside</option>
      <option value="outside">Outside</option>
    </select>
  </div>
  <div>
    <label>Features</label>
    <div>
      <input type="checkbox" value="blind" bind:group={features} />
      <label for="blind">Blind</label>
    </div>
    <div>
      <input type="checkbox" value="double-barrel" bind:group={features} />
      <label for="double-barrel">Double Barrel</label>
    </div>
    <div>
      <input type="checkbox" value="iaaf-certified" bind:group={features} />
      <label for="iaaf">IAAF Certified</label>
    </div>
    <div>
      <input type="checkbox" value="olympic" bind:group={features} />
      <label for="olympic">Olympic Venue</label>
    </div>
    <div>
      <input type="checkbox" value="interesting" bind:group={features}/>
      <label for="interesting">Interesting</label>
    </div>
  </div>
  <div>
    <button type="submit">Submit</button>
  </div>
  <div>
    <form name="output">
      <textarea bind:value={output} rows="20" cols="50"></textarea>
    </form>
  </div>
</form>
