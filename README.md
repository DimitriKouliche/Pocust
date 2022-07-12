# Pocust

A clever wordplay between locust and POC, this repository was created to host locust tests made on POCs.

## Execute the tests

You can run tests on staging environments by using 

```bash
make stress-tests app_name=your_app load_factor=traffic_multiplier headless
```
Example
```bash
make stress-tests app_name=mediaapi load_factor=1.5 headless=true
```

If you need to specify a host you can use 

```bash
make stress-tests app_name=your_app host=your_host load_factor=traffic_multiplier
```
Example
```bash
make stress-tests app_name=mediaapi host=http://localhost:8042 load_factor=1.5
```

## Build and distribute tests

**Building docker image**
```bash
make build
```

**Distribute docker image**
```bash
make distribute
```