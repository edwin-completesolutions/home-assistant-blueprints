alias: <Automation Name> v1 · <Short Purpose>
description: >
  <1-liner: wat doet het?>
  <Wanneer loopt het, bv. elke 10s of bij een trigger>
  <Welke hoofdfunctie voert het uit>

triggers:
  - trigger: time_pattern
    seconds: /20  # of een andere trigger
conditions: []
actions:
  - alias: Read sensors
    variables:
      sensor_control_enabled: "{{ is_state('input_boolean.control_enable', 'on') }}"
      sensor_power_w: "{{ states('sensor.nett_power') | float(0) }}"
      sensor_soc: "{{ states('sensor.battery_soc') | float(0) }}"

  - alias: Compute derived values
    variables:
      target_w: -50
      current_signed_w: |
        {% if is_state('select.mode', 'Discharge') %}
          {% set current_signed_w = states('number.output_w') | float(0) %}
        {% else %}
          {% set current_signed_w = (-1 * states('number.input_w') | float(0)) %}
        {% endif %}
        {{ current_signed_w }}
      desired_target_w: "{{ (current_signed_w - sensor_power_w + target_w) | round(0) }}"

  - alias: Apply limits and deadband → limited_w
    variables:
      deadband_w: 30
      max_charge_w: 1100
      max_discharge_w: 800
      limited_w: |
        {% set limited_w = desired_target_w %}
        {% if limited_w | abs <= deadband_w %}
          {% set limited_w = 0 %}
        {% endif %}
        {% if limited_w > 0 %}
          {% set limited_w = [limited_w, max_discharge_w] | min %}
        {% elif limited_w < 0 %}
          {% set limited_w = [limited_w, -max_charge_w] | max %}
        {% endif %}
        {{ limited_w }}

  - alias: Decide action and update text
    variables:
      state_text: |
        {% if limited_w == 0 %}
          standby
        {% elif limited_w > 0 %}
          discharging
        {% else %}
          charging
        {% endif %}
        {{ state_text }}

  - alias: Apply setpoints
    choose:
      - conditions:
          - condition: template
            value_template: "{{ limited_w > 0 }}"
        sequence:
          - action: number.set_value
            target:
              entity_id: number.outputlimit
            data:
              value: "{{ limited_w }}"
      - conditions:
          - condition: template
            value_template: "{{ limited_w < 0 }}"
        sequence:
          - action: number.set_value
            target:
              entity_id: number.inputlimit
            data:
              value: "{{ -1 * limited_w }}"

  - alias: Update status text
    if:
      - condition: template
        value_template: "{{ is_state('input_boolean.control_enable', 'on') }}"
    then:
      - action: input_text.set_value
        target:
          entity_id: input_text.state_desc
        data:
          value: "{{ state_text }}"

mode: restart

