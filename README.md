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

# Current Check-In

## REST-Sensor

This sensor returns the most recent Check-in with some details. For more options please refer to Träwelling API documentation.

        - method: GET
            resource: https://traewelling.de/api/v1/user/USERNAME/statuses?page=1
            headers:
            accept: application/json
            Authorisation: "API-KEY"
            X-CSRF-TOKEN: ""
            scan_interval: 90
            sensor:
            - name: Traewelling Check-In-ID
                value_template: >
                {{ value_json.data[0].id }}
                icon: mdi:train
                unique_id: traewelling_check_in_id
            - name: Traewelling Destination Name
                value_template: >
                {{ value_json.data[0].train.destination.name }}
                icon: mdi:train
                unique_id: traewelling_destination_name
            - name: Traewelling Arrival RealTime
                value_template: >
                {{ value_json.data[0].train.destination.arrivalReal }}
                icon: mdi:bus-clock
                unique_id: traewelling_arrival_realtime
            - name: Traewelling Origin Time
                value_template: >
                {{ value_json.data[0].train.origin.departurePlanned }}
                icon: mdi:bus-clock
                unique_id: traewelling_origin_plannedtime
            - name: Traewelling Line
                value_template: >
                {{ value_json.data[0].train.lineName }}
                icon: mdi:train
                unique_id: traewelling_line

## Binary Sensor

This binary template sensor named "Träwelling Check-In aktiv" flips to on if user is currently checked in (plus 5 minutes in the past and future).

    {% set departure = states('sensor.traewelling_origin_time') %}
    {% set arrival   = states('sensor.traewelling_arrival_realtime') %}
    {% if departure in ['unknown', 'unavailable'] or arrival in ['unknown','unavailable'] %}
            err
    {% else %}
    {% set dep = as_datetime(departure) - timedelta(minutes=5) %}
    {% set arr = as_datetime(arrival) + timedelta(minutes=5) %}
    {% set now = now() %}
    {{ now >= dep and now <= arr }}
    {% endif %}

## Dashboard card

This card shows current data on dashboard if user is checked-in as defined by the binary sensor.

    type: markdown
    content: >-
    Unterwegs mit {{states("sensor.traewelling_line")}} nach
    {{states("sensor.traewelling_destination_name")}}, ETA:
    {{as_timestamp(states("sensor.traewelling_arrival_realtime"))|timestamp_custom('%H:%M')}}

    Mehr Infos:
    https://traewelling.de/status/{{states("sensor.traewelling_check_in_id")}}
    visibility:
    - condition: state
        entity: binary_sensor.trawelling_check_in_aktiv
        state: "on"

# Check-in reminder

This automation reminds the user to check in if connecting to public transport wifi.

        alias: Träwelling check-in reminder
        description: ""
        triggers:
        - trigger: template
            value_template: >-
            {{ states('sensor.jelly_star_wi_fi_connection') in ['nordbahn-WiFi',
            'WIFI@DB', 'Mobyklick', 'Regio-S-Bahn WLAN', 'WLAN@start', 'WIFIonICE',
            'metronom free WLAN', 'LieblingsWlan@erixxSH', 'UESTRA_free_WIFI'] }}
            for:
            hours: 0
            minutes: 1
            seconds: 0
        conditions:
        - condition: state
            entity_id: binary_sensor.trawelling_check_in_aktiv
            state:
            - "off"
        actions:
        - action: telegram_bot.send_message
            metadata: {}
            data:
            message: Bitte in Träwelling einchecken!
        mode: single
