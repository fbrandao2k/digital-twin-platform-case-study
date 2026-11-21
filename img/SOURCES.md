# Image sources

The images come from two places: the demo recordings of November 2026, and the screenshots in the platform's own design notes.

## Frames from the demo recordings

These are frames or clips from the demo recordings of November 2026 (`ApresentacaoWebAppNov2026`, not published). Frames were cropped below the browser's address bar and above the Windows taskbar, then scaled to 1600 px wide (JPEG, quality 88).

| Image | Recording | Time |
| --- | --- | --- |
| `hero-gis.jpg` | `8-GIS_Integration.mp4` | 0:45 |
| `map-assistant.jpg` | `15-Map-Cesium_Assistant.mp4` | 1:25 |
| `smart-views.jpg` | `3-Properties_IFC_Custom.mp4` | 0:58 |
| `fem-displacements.jpg` | `6-FEM_CSIBridge_DisplacementStresses.mp4` | 1:05 |
| `fem-stresses.jpg` | `6-FEM_CSIBridge_DisplacementStresses.mp4` | 2:30 |
| `fem-modes.gif` | `7-FEM_Ansys_Modes.mp4` | 1:18.5 to 1:21.6, viewport only, 800 px, about 12 fps |
| `damage-assessment.jpg` | `11-DamageAssessment-Part1.mp4` | 1:00 |
| `risk.jpg` | `4-Risk_Module.mp4` | 0:35 |
| `sheets.jpg` | `9-GoogleDriveSheets_Integration.mp4` | 0:55 |
| `sensors.jpg` | `12-IoT_Sensors.mp4` | 0:26 |
| `ai-agent.jpg` | `14-AI-Agents_DamageAssessment.mp4` | 1:05 |

`ai-agent.jpg` has the "Key saved in this browser" line blurred, because it shows the last characters of the API key. The recording predates November 2026, when the agent still used each user's own key; it now calls Claude through a server-side function.

## Screenshots from the platform's design notes

These images come from the design notes in the platform's private repository. They were taken in the development app (or rendered from a generated IFC file) in November 2026. They are kept at their original resolution, never scaled up, so they are narrower than the recording frames. Three were cropped to the viewport without rescaling and saved as JPEG at quality 92; the other two are unchanged copies.

| Image | What it shows | Size | Edits |
| --- | --- | --- | --- |
| `ponte-minas-model.jpg` | Render of the generated Ponte de Minas IFC file (no scan points) | 1400 × 424 | none |
| `ponte-mecanica-gauges.jpg` | Ponte da Mecânica IFC in the viewer, with replayed strain-gauge markers on their IfcSensor elements (no readings visible) | 414 × 403 | none |
| `dam-demo-sensors.jpg` | The fictitious generated dam of the live-sensor demo, with its three markers not connected to a broker | 470 × 404 | cropped to the viewport, sensors panel removed |
| `point-cloud-rgb.jpg` | Simulated drone LiDAR scan of the demo dam (synthetic data), colored by RGB | 628 × 316 | cropped to the viewport |
| `point-cloud-height.jpg` | The same simulated scan, colored by height | 628 × 316 | cropped to the viewport |

## Before adding an image

Check that it shows none of these: Firebase project IDs, Firestore document IDs or Storage paths, AWS account IDs, ARNs or IoT endpoints (for example in the Broker URL field), console or address-bar URLs, local or network file paths, emails or names of real users, pages of inspection reports, API keys or their hints, recorded lab readings (for example, open strain-gauge cards), results of the lab's NeRF comparison study, or bridge details an owner has not agreed to publish.
