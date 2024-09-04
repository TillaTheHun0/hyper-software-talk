---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: hyper Software
info: |
  Ideas on how to build software to minimize technical debt and
  scale on your terms

# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  enabled: true
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# hyper Software

Ideas on how to build software to <br />
Minimize technical debt & <br />
Scale on your terms

---

# Roadmap

- Lingua Franca
- Introduction to "Architecture"
- Patterns & examples for building hyper Software
  - Inversion of Control
  - Stratified Design
  - Open-Closed Principle
  - Ports & Adapters ✨⚡️

---

# Lingua Franca

<div v-click>
<b>Abstraction</b>: a generalization of a piece of data or operation. A layer of indirection.
</div>

<div v-click>
<b>Encapsulation</b>: an abstraction by way of "hiding" <i>all</i>, but the relevant features, about a piece of data or operation
</div>

<br/>
<br/>

<div v-click>
<b>We can have an <i>abstraction</i> that does <i>not encapsulate</i> well aka. a "leaky" abstraction</b>
</div>

<br/>

---

# Lingua Franca

<div v-click>
<b>Entity/Model</b>: data, represented by a shape, and a set of rules on how to enforce/modify that shape
</div>

<div v-click>
<b>Business Logic</b>: the real-world bonafide rules of the business, implemented in code

<li><b>NO side-effects</b></li>
<li><b>SEPARATE from any external service aka. Database</b></li>
</div>

<div v-click>
<b>Side-Effect</b>: anything non-deterministic or that depends on the environment external from the business logic. Simply put:
<li><b>I/O</b> -- databases, queues, caches, buckets, 3rd party SaaS, etc...</li>
<li>Presentation</li>
</div>

<div v-click>
<b>Domain</b>: an abstraction (ideally an encapsulation) of a specific business context and accompanying business rules aka. a "bounded context".

<li><b>Not to be confused with a "web" domain</b></li>
<li>Examples: "Billing" "Profile" "Notifications" (NOT 1-1 with Models)</li>
</div>

<div v-click>
<b>We can have a Domain that does not <i>encapsulate</i> well aka. a "leaky" Domain</b>
</div>

---
layout: center
---
# How can we tell if an API is "leaky"?

---
layout: image-right
image: /wheel.webp
backgroundSize: contain
---

<div v-click>

- Made of rubber, plastic, sometimes wood
- Actuates a rack and pinion or hydraulic system
- OR sends a signal to the ECU
- ~18" in diameter
- What about for a go-cart?
- What about for a plane?
- etc...

<br />OR...

</div>

<div v-click>

- It makes the vehicle change/maintain direction

</div>

<br />

<div v-click>
<b>A steering wheel is an <i>encapsulation</i> of how to make the car change direction</b>
</div>

---
layout: image-right
image: /switch.jpg
backgroundSize: contain
---

<div v-click>

- Made of plastic, wood, or metal
- Completes an electrical circuit for a 12 gauge wire, running on a 15 amp circuit
- Circuit connects to 120 volt phase, on a 240 volt split-phase system.
- A/C sent from a power plant within 50 miles...
- Etc.

<br />OR...

</div>

<div v-click>

- It makes the light turn on or off

</div>

<br />

<div v-click>
<b>A light-switch is an <i>encapsulation</i> of how to send power to an electrical component</b>
</div>

---
layout: center
---

## An API is leaky when:
<div v-click>
- In order to use it, the consumer needs to know how it works.
</div>
<div v-click>
- It exposes more than what the consumer needs to know
</div>
<div v-click>
- Change in implementation requires the consumer to change, <b>even if their goal hasn't changed</b>
</div>

---

# What is Software Architecture?

- Patterns for structuring software modules
- MVC, Event-Driven, Event Sourced, Ports and Adapters, Onion
- Put this file here, put this file there...
- Use React/Svelte/Vue/Next/Remix/SvelteKit/Deno/...
- Use JS/TS/Rust/Go/Erlang/Assembly

...OR

<div v-click>
<b>It allows for modifying one piece of the software without breaking another, unrelated, piece of software</b>

A key quality indicator of a piece of software is its ability to be
modified without breaking something unrelated.

The most important thing to encapsulate...
</div>

<div v-click>
<b>Business Logic</b>

We ought to use patterns and tools that help us encapsulate our business logic
and make it easy to understand and **change over time**
</div>

---
zoom: 0.65
---

# Inversion of Control aka. Dependency Injection

- Give code it's dependencies
- _`inject`_ side-effects, _`import`_ pure-utils & business logic

<!-- No dependency injection -->
````md magic-move {lines: true}
```js{*}
import { differenceInYears } from 'date-fns'

import { Trip } from './db/Trip.js'
import { Driver } from './db/Driver.js'

async function calcTotalUnderageMileage ({ carId }) {
  /**
   * find all trips taken with the car
   */
  const allTrips = await Trip.findAll({ carId })

  /**
   * find all drivers for each Trip
   */
  const driverIds = allTrips.flatMap((t) => t.drivers)
  const allDrivers = await Driver.findAll({ id: { $in: driverIds }})

  /**
   * Find the intersection of trips and drivers
   * that were underage, at the time of the trip
   */
  const underageTrips = allTrips.reduce(
    (acc, trip) => {
      const startDate = trip.startDate
      const drivers = allDrivers.filter((d) => trip.drivers.find(id => id === d.id)) 

      const wasUnderage = drivers.find((d) => differenceInYears(startDate, d.birthDate) < 18)
      if (wasUnderage) acc.push(trip)

      return acc
    },
    []
  )

  const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
  return mileage
}
```

<!-- One-One dependency injection -->
```js{*}
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ Trip, Driver }) {

 return async function calcTotalUnderageMileage ({ carId }) {
    /**
     * find all trips taken with the car
     */
    const allTrips = await Trip.findAll({ carId })

    /**
     * find all drivers for each Trip
     */
    const driverIds = allTrips.flatMap((t) => t.drivers)
    const allDrivers = await Driver.findAll({ id: { $in: driverIds }})

    /**
     * Find the intersection of trips and drivers
     * that were underage, at the time of the trip
     */
    const underageTrips = allTrips.reduce(
      (acc, trip) => {
        const startDate = trip.startDate
        const drivers = allDrivers.filter((d) => trip.drivers.find(id => id === d.id)) 

        const wasUnderage = drivers.find((d) => differenceInYears(startDate, d.birthDate) < 18)
        if (wasUnderage) acc.push(trip)

        return acc
      },
      []
    )

    const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
    return mileage
  }
}
```

<!-- Complete dependency injection -->
```js{*}
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {

 return async function calcTotalUnderageMileage ({ carId }) {
    const { trips: allTrips, drivers: allDrivers } = await findTripsWithDriversByCarId(carId)

    /**
     * Find the intersection of trips and drivers
     * that were underage, at the time of the trip
     */
    const underageTrips = allTrips.reduce(
      (acc, trip) => {
        const startDate = trip.startDate
        const drivers = allDrivers.filter((d) => trip.drivers.find(id => id === d.id)) 

        const wasUnderage = drivers.find((d) => differenceInYears(startDate, d.birthDate) < 18)
        if (wasUnderage) acc.push(trip)

        return acc
      },
      []
    )

    const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
    return mileage
  }
}
```
````

---
zoom: 0.65
---

# Easy to test!

````md magic-move {lines: true}
```js{*}
import { describe, test } from 'node:test'
import assert from 'node:assert'

import { subYears, subMonths } from 'date-fns'

import { calculateTotalMileageWith } from './TripManager.js'

describe('calcTotalUnderageMileage', () => {
  describe('should calculate the total underage mileage', () => {
    const CAR_ID = 'car-123'
    const [D1, D2, D3] = ['123', '456', '789']
    const TRIPS = [
      { startDate: subMonths(new Date(), 6), miles: 1000, drivers: [D1, D2] },
      { startDate: subMonths(new Date(), 4),  miles: 300, drivers: [D2] },
      { startDate: subMonths(new Date(), 18), miles: 250, drivers: [D3] }
    ]

    test('when there are underage drivers and trips', async () => {
      const calculateTotalMileage = calculateTotalMileageWith({
        findTripsWithDriversByCarId: async (carId) => {
          // Ensure we pass the correct parameters to the dep
          assert.equals(carId, CAR_ID)

          return {
            trips: TRIPS,
            drivers: [
              { id: D1, birthDate: subYears(new Date(), 15) },
              { id: D2, birthDate: subYears(new Date(), 35) },
              { id: D3, birthDate: subYears(new Date(), 18) },
            ]
          }
        }
      })

      const res = await calculateTotalMileage({ carId: CAR_ID })
      assert.equals(res, 1250)
    })
  })
})
```

```js{*}
import { describe, test } from 'node:test'
import assert from 'node:assert'

import { subYears, subMonths } from 'date-fns'

import { calculateTotalMileageWith } from './TripManager.js'

describe('calcTotalUnderageMileage', () => {
  describe('should calculate the total underage mileage', () => {
    const CAR_ID = 'car-123'
    const [D1, D2, D3] = ['123', '456', '789']
    const TRIPS = [
      { startDate: subMonths(new Date(), 6), miles: 1000, drivers: [D1, D2] },
      { startDate: subMonths(new Date(), 4),  miles: 300, drivers: [D2] },
      { startDate: subMonths(new Date(), 18), miles: 250, drivers: [D3] }
    ]

    test('when there are no underage drivers', async () => {
      const calculateTotalMileage = calculateTotalMileageWith({
        findTripsWithDriversByCarId: async (carId) => {
          assert.equals(carId, CAR_ID)

          return {
            trips: TRIPS,
            drivers: [
              { id: D1, birthDate: subYears(new Date(), 40) },
              { id: D2, birthDate: subYears(new Date(), 35) },
              { id: D3, birthDate: subYears(new Date(), 65) },
            ]
          }
        }
      })

      const res = await calculateTotalMileage({ carId: CAR_ID })
      assert.equals(res, 0)
    })
  })
})
```

```js{*}
import { describe, test } from 'node:test'
import assert from 'node:assert'

import { subYears, subMonths } from 'date-fns'

import { calculateTotalMileageWith } from './TripManager.js'

describe('calcTotalUnderageMileage', () => {
  describe('should calculate the total underage mileage', () => {
    const CAR_ID = 'car-123'
    const [D1, D2, D3] = ['123', '456', '789']
    const TRIPS = [
      { startDate: subMonths(new Date(), 6), miles: 1000, drivers: [D1, D2] },
      { startDate: subMonths(new Date(), 4),  miles: 300, drivers: [D2] },
      { startDate: subMonths(new Date(), 18), miles: 250, drivers: [D3] }
    ]

    test('when there are no trips', async () => {
      const calculateTotalMileage = calculateTotalMileageWith({
        findTripsWithDriversByCarId: async (carId) => {
          assert.equals(carId, CAR_ID)

          return { trips: [], drivers: [] }
        }
      })

      const res = await calculateTotalMileage({ carId: CAR_ID })
      assert.equals(res, 0)
    })
  })
})
```

````

---
zoom: 0.65
---

# How can I be sure that my stubs are always correct?

Use a Contract!

```js {*}{lines: true}
import { z } from 'zod'

const idSchema = z.string().min(1)
const tripSchema = z.object({
  // ....
  startDate: z.date(),
  miles: z.number().positive(),
  drivers: z.array(idSchema)
})
const driverSchema = z.object({
  // ...
  birthDate: z.date()
})

/**
 * Enforce inputs and outputs of the dependency
 */
const findTripsWithDriversByCarIdSchema = z.function()
  .args(idSchema)
  .returns(z.promise(
    z.object({
      trips: z.arrary(tripsSchema),
      drivers: z.array(driverSchema)
    })
  ))

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {
  /**
   * Wrap the dependency with a contract
   */
  findTripsWithDriversByCarId = findTripsWithDriversByCarIdSchema.implement(findTripsWithDriversByCarId)

 return async function calcTotalUnderageMileage ({ carId }) {
    // ...
    findTripsWithDriversByCarId(123) // ERROR wrong input type
    findTripsWithDriversByCarId('123') // returns { foobar } -> ERROR wrong output type
  }
}
```

---

# Stratified Design

Shifting imperative constructs _down_ and declarative constructs _up_

- The lower the level, the more _imperative_. The higher the level, the more _declarative_
- Business logic entrypoints should be declarative, like SQL

<img src='/stratified.jpg' />

---
zoom: 0.68
---

# Let's Stratify!

````md magic-move {lines: true}
```js{*}
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {

 return async function calcTotalUnderageMileage ({ carId }) {
    const { trips: allTrips, drivers: allDrivers } = await findTripsWithDriversByCarId(carId)

    /**
     * Find the intersection of trips and drivers
     * that were underage, at the time of the trip
     */
    const underageTrips = allTrips.reduce(
      (acc, trip) => {
        const drivers = allDrivers.filter((d) => trip.drivers.find(id => id === d.id)) 

        const wasUnderage = drivers.find((d) => differenceInYears(trip.startDate, d.birthDate) < 18)
        if (wasUnderage) acc.push(trip)

        return acc
      },
      []
    )

    const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
    return mileage
  }
}
```

```js{*}
import { curry, propSatisfies, flip, includes, filter } from 'ramda'
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {

 return async function calcTotalUnderageMileage ({ carId }) {
    const { trips: allTrips, drivers: allDrivers } = await findTripsWithDriversByCarId(carId)

    /**
     * Find the intersection of trips and drivers
     * that were underage, at the time of the trip
     */
    const underageTrips = allTrips.reduce(
      (acc, trip) => {
        const drivers = idIncludedIn(trip.drivers, allDrivers)

        const wasUnderage = drivers.find((d) => differenceInYears(trip.startDate, d.birthDate) < 18)
        if (wasUnderage) acc.push(trip)

        return acc
      },
      []
    )

    const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
    return mileage
  }
}

// Utils
const includedIn = flip(includes)
const idIncludedIn = curry((ids, objs) =>
  filter(propSatisfies(includedIn(ids), 'id'), objs)
)
```

```js{*}
import { curry, pipe, propSatisfies, flip, includes, filter} from 'ramda'
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {

 return async function calcTotalUnderageMileage ({ carId }) {
    const { trips: allTrips, drivers: allDrivers } = await findTripsWithDriversByCarId(carId)

    /**
     * Find the intersection of trips and drivers
     * that were underage, at the time of the trip
     */
    const underageTrips = allTrips.reduce(
      (acc, trip) => {
        const drivers = idIncludedIn(trip.drivers, allDrivers)

        const wasUnderage = drivers.find(wasUnderageAt(trip.startDate))
        if (wasUnderage) acc.push(trip)

        return acc
      },
      []
    )

    const mileage = underageTrips.reduce((acc, trip) => acc + trip.miles, 0)
    return mileage
  }
}

// Driver Specific
const ageAt = curry((date, driver) => differenceInYears(date, driver.birthDate))
const wasUnderageAt = pipe(ageAt, (age) => age < 18)

// Utils
const includedIn = flip(includes)
const idIncludedIn = curry((ids, objs) =>
  filter(propSatisfies(includedIn(ids), 'id'), objs)
)
```

```js{*}
import { curry, pipe, propSatisfies, filter, flip, includes, map, prop, sum, any } from 'ramda'
import { differenceInYears } from 'date-fns'

function calculateTotalMileageWith ({ findTripsWithDriversByCarId }) {

  /**
   * Declarative business logic 💯
   */
 return async function calcTotalUnderageMileage ({ carId }) {
    const { trips, drivers } = await findTripsWithDriversByCarId(carId)

    return pipe(collectUnderageTrips, sumMiles)({ trips, drivers })
  }
}

// Trip Specific
const collectUnderageTrips = ({ trips, drivers }) => filter(
  (trip) => pipe(
    idIncludedIn(trip.drivers),
    any(wasUnderageAt(trip.startDate))
  )(drivers)
  trips
)
const sumMiles = pipe(map(prop('miles')), sum)

// Driver Specific
const ageAt = curry((date, driver) => differenceInYears(date, driver.birthDate))
const wasUnderageAt = pipe(ageAt, (age) => age < 18)

// Utils
const includedIn = flip(includes)
const idIncludedIn = curry((ids, objs) =>
  filter(propSatisfies(includedIn(ids), 'id'), objs)
)
```
````

---
layout: image-right
image: /ioda.png
backgroundSize: contain
zoom: 0.8
---

# Open-Closed Principle

functionality should be closed for modification and open for extension

AKA.

<div v-click>
<b>Composition!</b>

Not inheritance -- inheritance sucks.
</div>

<div v-click>
<b>Open-Closed is encouraged by Stratified Design!</b>

Strata within Strata within Strata -- Strata Composition!
</div>

<br />
<br />
<br />

<div v-click>
<b>All of this culminates in...</b>
</div>

---
layout: image-right
image: /hex.png
backgroundSize: contain
zoom: 0.8
---
# Ports & Adapters

- `Port`: contract _into_ or _out of_ the business layer (domain)
  - **Nothing** enters/exits the business layer without going through a port
- `Adapter`: perform the actual communication between external actors and the business layer
  - Driving `Adapter`: calls into the business layer, by way of a `Port`. ie. Presentation
  - Driven `Adapter`: called by the business layer, by way of a `Port` to interact with some side-effect, ie.
    - database
    - storage bucket
    - cache
    - queue
    - email
    - etc.
    - **Driven adapters implement the Port**

- **Dependencies go _inward_**

---
layout: center
---

```
Driving Adapter <--> Port <--> Business Layer <--> Port <--> Driven Adapter
```

---

# Data

<div v-click>
<code>RESTful api -> Core -> Data Port -> CouchDB Adapter</code>

OR...
</div>


<div v-click>
<code>GraphQL api -> Core -> Data Port -> Postgres Adapter</code>

OR...
</div>


<div v-click>
<code>CLI -> Core -> Data Port -> DynamoDB Adapter</code>
</div>

---

# Cache

<div v-click>
<code>GRPC api -> Core -> Cache Port -> Redis Adapter</code>

OR...
</div>


<div v-click>
<code>GraphQL api -> Core -> Cache Port -> Sqlite Adapter</code>

OR...
</div>


<div v-click>
<code>CLI -> Core -> Cache Port -> FlatFile Adapter</code>
</div>

---

# Queue

<div v-click>
<code>GRPC api -> Core -> Queue Port -> Sqlite Adapter</code>

OR...
</div>


<div v-click>
<code>GraphQL api -> Core -> Queue Port -> RabbitMQ Adapter</code>

OR...
</div>


<div v-click>
<code>CLI -> Core -> Queue Port -> BullMQ Adapter</code>
</div>

<br/>

<div v-click>
<b>Adapters are interchangeable</b>
</div>

---
layout: image-right
image: /hover-lightning.svg
---

# `hyper` Service Framework

- Ports and Adapters, as a Services Tier
- use `hyper-nano` when developing locally
- Deploy a `hyper` `Server` in an environment
  - Pick the adapters for your underlying services

---

# Closing Thoughts

<div v-click>
<li>Don't start with microservices -- humans are really bad a predicting the future</li>
</div>

<div v-click>
<li>Modular Monolith FTW -- Ports and Adapters w/ <i>Logically Separated</i> Domains. Single Deployable</li>
</div>

<div v-click>
<li>Trunk Based Development -- deploy continuously, use Feature Flags for release management</li>
</div>

<div v-click>
<li>Build something -- then assume that its <b>wrong!</b></li>
</div>

<br/>
<br/>
<br/>

<div v-click>
<i>When building software, every design decision ought to be understood as accepting a set of constraints. Therefore, we ought to defer accepting constraints until we have as much information as possible. Clean Architecture enables us to do just that, without causing a "blast radius" effect across the codebase, when we make the wrong decision, or inevitably need to change something.</i>
</div>
