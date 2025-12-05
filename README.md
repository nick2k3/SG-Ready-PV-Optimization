<!DOCTYPE html>
<html lang="de">
<body>
  <h1>⚡ SG Ready PV-Optimierung Blueprint für Home Assistant ⚡</h1>

<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fchino-lu%2FSG-Ready-PV-Optimierung%2Frefs%2Fheads%2Fmain%2Fsgready.yaml" target="_blank" rel="noreferrer noopener"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a>
  <br><br>
  <p>
    <span class="emoji">📋</span>
    <b>Dieses Blueprint automatisiert die Umschaltung der SG Ready-Eingänge (SG0 und SG1) deiner Wärmepumpe abhängig vom PV-Überschuss (z. B. bei Kostal Wechselrichtern) – im Automatik- oder manuellem Modus per Dashboard-Auswahl.</b>
  </p>

  <h2>🔍 Funktionsumfang</h2>
  <ul>
    <li>🤖 <b>Automatisches Schalten</b> von SG0 und SG1 je nach PV-Überschuss (NEGATIV-Werte bei Einspeisung)</li>
    <li>🖲️ <b>Manuelle Auswahl</b> zwischen Normbetrieb, PV-Optimierung, PV-Boost und Netzsperre per input_select</li>
    <li>⚙️ <b>Konfigurierbare Schwellwerte</b> für jeden Zustand (SG0, SG1, AUS)</li>
    <li>⏱️ <b>Schaltverzögerung/Hysterese</b> über einen einstellbaren Zeitraum (z. B. 60 Sekunden)</li>
    <li>🚫 Kein EVU Sperre-Helper mehr notwendig</li>
    <li>📱 Einfache Integration ins Home Assistant Dashboard (z. B. mit Mushroom-Cards)</li>
    <li>🔗 <b>Import per Button!</b></li>
  </ul>

  <h2>⚡ Schnelleinstieg</h2>
  <h3>1️⃣ Blueprint importieren</h3>
  <p>
    <a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fchino-lu%2FSG-Ready-PV-Optimierung%2Frefs%2Fheads%2Fmain%2Fsgready.yaml" target="_blank" rel="noreferrer noopener"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a>
  </p>

  <h3>2️⃣ Voraussetzungen</h3>
  <ul>
    <li>🏠 Home Assistant Installation (Version 2022.5 oder neuer empfohlen)</li>
    <li>🟩 Zwei Relais/Switches für SG0 und SG1 (z. B. Shelly, ESPHome, Sonoff)</li>
    <li>🔌 PV-Sensor mit NEGATIVEM Wert bei Einspeisung (z. B. Kostal Grid Power)</li>
    <li>🎚️ <b>input_select</b>-Helfer zur Modus-Auswahl (über Einstellungen → Helfer → Auswahlliste)</li>
  </ul>
  <p>
    <b>Beispiel für den input_select (configuration.yaml):</b>
    <pre><code>input_select:
  sg_ready_modus:
    name: SG Ready Modus
    options:
      - "Auto"
      - "Normal operation"
      - "PV-Optimization"
      - "PV-Boost"
      - "Network block"
    initial: "Auto"
    icon: mdi:heat-pump
</code></pre>
    <span class="emoji">💡</span> Oder lege ihn einfach als „Auswahlliste“ in den Home Assistant Einstellungen an.
  </p>

  <h3>3️⃣ Automatisierung mit Blueprint anlegen</h3>
  <ul>
    <li>🔧 Nach dem Import findest du das Blueprint unter <b>Einstellungen → Automatisierungen &amp; Szenen → Blueprints</b></li>
    <li>➕ Erstelle eine neue Automatisierung auf Basis dieses Blueprints</li>
    <li>⚡ Wähle deine Entitäten:
      <ul>
        <li>🔋 PV-Sensor (NEGATIV bei Einspeisung)</li>
        <li>🔌 Relais/Schalter für SG0 und SG1</li>
        <li>🖲️ Den input_select für die Moduswahl</li>
        <li>⚙️ Schwellenwerte und Verzögerung anpassen (siehe Tipps unten)</li>
      </ul>
    </li>
  </ul>

  <h3>4️⃣ Dashboard/Visualisierung (Empfehlung)</h3>
  <p>Für eine smarte Bedienung im Dashboard empfehlen wir <b>Mushroom Cards</b> mit einer Chips-Card (Icons als Tasten) und einer Status-Anzeige-Card:</p>

  <pre><code>type: custom:mushroom-chips-card
alignment: center
chips:
  - type: template
    icon: mdi:robot
    icon_color: "{{ 'blue' if is_state('input_select.sg_ready_modus','Auto') else 'grey' }}"
    tap_action:
      action: call-service
      service: input_select.select_option
      data:
        option: "Auto"
      target:
        entity_id: input_select.sg_ready_modus
    tooltip: Auto-Modus
  # ... weitere Chips für die Modi
</code></pre>
  <p>Für die aktuelle Statusanzeige (welcher Modus ist aktiv):</p>
  <pre><code>type: custom:mushroom-template-card
primary: &gt;
  {{ 
    iif(is_state('input_select.sg_ready_modus','Auto'),
      (
        iif(is_state('switch.sg0','off') and is_state('switch.sg1','off'), 'Normbetrieb',
        iif(is_state('switch.sg0','on') and is_state('switch.sg1','off'), 'PV-Optimierung',
        iif(is_state('switch.sg0','on') and is_state('switch.sg1','on'), 'PV-Boost',
        iif(is_state('switch.sg0','off') and is_state('switch.sg1','on'), 'Netzsperre', 'Unbekannt'))))
      ),
      states('input_select.sg_ready_modus')
    ) 
  }}
icon: mdi:heat-pump
fill_container: true
multiline_primary: true
</code></pre>
  <p>
    <span class="emoji">🧩</span> <i>Für Mushroom Cards benötigst du <a href="https://hacs.xyz/" target="_blank">HACS</a> und <a href="https://github.com/piitaya/lovelace-mushroom" target="_blank">Mushroom Cards</a>.</i>
  </p>

  <h2>💡 Tipps zur Einrichtung</h2>
  <ul>
    <li>➖ <b>NEGATIVE Schwellenwerte:</b> Je niedriger der Wert, desto mehr PV-Überschuss (z. B. SG0: -1000, SG1: -3000, Aus: -500)</li>
    <li>⏳ <b>Delay/Schaltverzögerung:</b> Dämpft das Schalten bei kurzfristigen Schwankungen. 30-60 Sekunden ist ein bewährter Wert.</li>
    <li>🎚️ <b>input_select:</b> Stelle sicher, dass der input_select die korrekten Optionen hat (siehe YAML oben).</li>
    <li>🧪 <b>Testen:</b> Prüfe nach der Einrichtung, ob die Relais wie gewünscht schalten (sowohl automatisch als auch manuell).</li>
    <li>🔧 <b>Anpassbar:</b> Du kannst jederzeit neue Optionen im input_select ergänzen, solange sie im Blueprint berücksichtigt sind.</li>
  </ul>

  <h2>❓ FAQ</h2>
  <p><b>Mein Sensor hat POSITIVE Werte bei Einspeisung – was tun?</b><br>
  ➡️ Dann musst du die Schwellenlogik umdrehen (im Blueprint die Schwellwerte und Bedingungen anpassen).</p>

  <p><b>Warum werden meine Relais zu oft geschaltet?</b><br>
  ➡️ Erhöhe die Verzögerung (<code>delay</code>) oder arbeite mit einem gleitenden Mittelwert-Sensor für mehr Stabilität.</p>

  <p><b>Kann ich weitere Betriebsmodi ergänzen?</b><br>
  ➡️ Ja, passe dafür einfach deinen input_select und die Action-Logik im Blueprint an.</p>

  <h2>💬 Support &amp; Feedback</h2>
  <p>Gerne Featurewünsche oder Probleme als <b>GitHub Issue</b> einstellen.<br>Pull Requests willkommen! 🙏</p>

  <h2>📝 Lizenz</h2>
  <p>MIT License</p>

  <hr>
  <p><b>🌞 Viel Spaß beim Optimieren deiner Wärmepumpe! 🔥</b></p>
</body>
</html>
