# Traewelling-Home-Assistant
Some Templates to include Träwelling data into Home Assistant

# Sensor Total Distance of User

This sensor shows the total träwell distance of a given user. Can be used to feed a utility meter to create statistics.

    rest:
      - method: GET
        resource: https://traewelling.de/api/v1/user/USERNAME
        headers:
          accept: application/json
          Authorisation: "API-KEY"
          X-CSRF-TOKEN: ""
        scan_interval: 900
        sensor:
          - name: Bahn Strecke
            value_template: >
              {{ (value_json.data.totalDistance / 1000) | round(1) }}
            state_class: measurement
            unit_of_measurement: "km"
            icon: mdi:train
            unique_id: traewelling_distance
