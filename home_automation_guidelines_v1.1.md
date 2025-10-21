# í ¾í·­ Home Assistant Automation Style Guide (v1.1)

### í ¼í¾¯ Purpose
Establish a clear, consistent, and maintainable format for Home Assistant automations written in YAML â€” with explicit aliasing, variables, version control, and structured logic flow.

---

## í ½í·‚ 0. Automations inside Packages (separate YAML file)

When placing automations inside a package file (via `packages:` or `!include_dir_named`), apply these extra rules:

### 0.1 Root key & indentation
Start with the root section:

    automation:
      - id: "<unique_stable_id>"
        alias: <Short name> vX Â· <main action>
        description: >
          <3â€“4 lines: what it does, when it runs, what it affects>
        triggers:
          # ...
        conditions: []
        actions:
          # ...
        mode: restart

Indentation:
- `automation:` is the root key.
- `- id:` begins the list (2 spaces deeper).
- All nested fields are 2 spaces deeper again.

### 0.2 Required id
- Every automation must include a unique stable id.
- Use kebab-case or snake_case with only ASCII characters.
  Examples:  
  `id: "solarflow-zero-on-meter"` or `id: "solarflow_zero_on_meter"`.
- The id must remain stable across all future versions.
- Each id must be globally unique within Home Assistant.

### 0.3 Comments allowed (in packages only)
- Comments are encouraged in package files for readability and debugging.
  Example:

      #â€” Parameters â€”#
      # Deadband Â±50 W to prevent flip/flop behavior

- Keep comments concise, meaningful, and relevant.

### 0.4 File name vs id
- File names do not affect automation identity.
- You can rename the YAML file without breaking anything.
- The id defines the automationâ€™s permanent link to HA.
- Use filenames only for organization.

### 0.5 Enable/disable in UI
- Automations in packages can be toggled in the UI; disabling works fully.
- Deleting from UI does not remove the YAML file. Manage files manually.

---

## í ¾í·± 1. Base structure for automations

Always use this layout:

    alias: <Short name> vX Â· <core function>
    description: >
      <Brief summary of what, when, and why>
    triggers:
      # ...
    conditions: []
    actions:
      # ...
    mode: restart

In package files, this section is nested under the automation: key.

---

## í ¾íº¶ 2. Aliases

Each major logical step in the automation should have an alias line.

Examples:
- alias: Read sensors
- alias: Compute derived values
- alias: Apply limits and guards
- alias: Apply setpoints

Use descriptive verbs. Avoid abbreviations except for units (_w, _pct, _s).

---

## í ¾í·© 3. Variables

- Define variables explicitly with a variables: block.
- Use snake_case naming and one variable per line.
- Add a unit suffix where relevant.

Example:

    variables:
      control_enabled: "{{ is_state('input_boolean.control_enable', 'on') }}"
      grid_power_w: "{{ states('sensor.grid_power') | float(0) }}"

---

## âš™ï¸ 4. Jinja Style

- Always quote Jinja expressions: "{{ ... }}"
- Use multi-line style with a pipe (|) for readability.
- Avoid inline if statements on one line.

Example:

    variables:
      target_w: |
        {% set diff = (grid_w + load_w) | int(0) %}
        {{ 0 - diff }}

---

## í ¾í·® 5. Action Flow

Structure actions in clear logical stages:

1. Read sensors and inputs  
2. Compute derived values  
3. Apply limits, caps, hysteresis, and guards  
4. Decide target values  
5. Apply outputs (services)  
6. Update UI helpers or status text

---

## í ¾í·  6. If / Else Logic

- Use full if / elif / else blocks for logic clarity.
- Use choose: when you need multiple distinct action paths.

---

## í ¾í·± 7. Constants and Parameters

- Keep constants local to where they are used.
- For shared parameters, use input_number.* helpers with descriptive names.

---

## í ¾í·¾ 8. Description Field

- Use block style (>) for multi-line descriptions.
- First line: short summary.
- Following lines: interval, purpose, or safety notes.

Example:

    description: >
      Runs every 20s. Keeps net export near zero by charging/discharging dynamically.
      Applies SoC limits, deadband, and discharge-allowed guard.

---

## í ¾í·° 9. Services and Updates

- Always specify both target and data fields explicitly.

Example:

    - action: number.set_value
      target:
        entity_id: number.target_power_w
      data:
        value: "{{ desired_w }}"

---

## í ¾í·© 10. Debug and Status

- Use helpers like input_text.solarflow_state_desc for human-readable status.
- Keep output text concise, e.g. â€œCharging blocked â€“ SoC too highâ€.

---

## í ½í³ 11. Formatting Rules

- Use two spaces per indent level, no tabs.
- Compare states using 'on' / 'off'.
- Keep numbers unquoted unless used inside strings.
- Separate logical sections with blank lines.

---

## í ¾í·© 12. Versioning and ID Policy

- Include version number in alias (v1, v2, v3, ...).
- Keep id stable across all versions.
- Optionally list brief changes in the description.

---

## âœï¸ 13. Mini Templates

### 13.1 Example: minimal package automation that reads and conditionally updates a helper

    automation:
      - id: "solarflow_discharge_allowed_sync"
        alias: SolarFlow Â· Discharge Allowed Sync v1
        description: >
          Reads current helper state, computes new value, and updates only if changed.
          Runs hourly and on relevant entity changes.

        triggers:
          - trigger: time_pattern
            hours: /1
          - trigger: state
            entity_id:
              - input_number.some_threshold
              - input_number.other_threshold

        conditions: []

        actions:
          - alias: Read current + compute new value
            variables:
              current_state: "{{ states('input_boolean.solarflow_discharge_allowed') }}"
              computed_should_allow: |
                {% set a = states('input_number.some_threshold') | int(0) %}
                {% set b = states('input_number.other_threshold') | int(0) %}
                {{ 'on' if a > b else 'off' }}

          - alias: Update helper only if changed
            choose:
              - conditions: "{{ current_state != computed_should_allow }}"
                sequence:
                  - action: input_boolean.turn_{{ 'on' if computed_should_allow == 'on' else 'off' }}
                    target:
                      entity_id: input_boolean.solarflow_discharge_allowed
            default: []
        mode: restart

### 13.2 Comment Pattern inside Packages

    #â€” Read sensors â€”#
    # Keep comments short, functional, and relevant.

---

## âœ¨ Summary

Packages:
- Start with "automation:".
- Always include a stable id.
- Maintain correct indentation.
- Comments are allowed but concise.

General:
- One clear intent per alias.
- Consistent Jinja syntax and variable naming.
- Structured, readable logic.
- Version noted in alias.
- Description explains purpose and timing.

---

âœ… Checklist for Reviews
- [ ] Starts with automation: (if in package)
- [ ] Includes stable id
- [ ] Correct indentation (2 spaces)
- [ ] Clear aliases and variables
- [ ] Version number in alias
- [ ] Description covers function and timing
- [ ] Comments concise and relevant
- [ ] Output helpers updated correctly
- [ ] Jinja readable and consistent
