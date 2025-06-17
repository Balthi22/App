/**
 * BS-ALARM Backend – Gruppen- und Userverwaltung, Sound-Upload, Admin-Rollen (Ready-to-Run)
 *
 * Starte mit:
 * 1. npm install express cors multer
 * 2. mkdir-Uploads
 * 3. Knoten app.js
 *
 * API läuft auf http://localhost:4000
 */
const express = erfordern('express');
const cors = erfordern('cors');
const multer = require('multer');
const fs = erfordern('fs');
const app = express();
konstanter PORT = 4000;

// Sicherstellen, dass das Upload-Verzeichnis vorhanden ist
if (!fs.existsSync('uploads')) {
  fs.mkdirSync('Uploads');
}

app.use(cors());
app.use(express.json());

// In-Memory-"Datenbank"
lass Benutzer = [
  { id: 1, name: "Admin", email: "admin@bs-alarm.de", role: "admin", groupIds: [1] }
];
lass Gruppen = [
  { id: 1, name: "Feuerwehr" }
];
lass alarme = [];
lass Antworten = [];
lass klingt = [];

// Multer Storage für Sounddateien
const storage = multer.diskStorage({
  Ziel: Funktion (req, file, cb) {
    cb(null, 'uploads/');
  },
  Dateiname: Funktion (req, file, cb) {
    cb(null, Date.now() + '-' + file.originalname.replace(/[^a-zA-Z0-9.\-_]/g, '_'));
  }
});
const upload = multer({ storage });

/** Benutzer-API **/
// Neuen Benutzer anlegen (nur Admin)
app.post('/api/users', (req, res) => {
  const { Name, E-Mail, Rolle, Gruppen-IDs, Anfordererrolle } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  if (!name || !email) return res.status(400).json({ error: "Name und E-Mail erforderlich" });
  if (users.find(u => u.email === email)) return res.status(400).json({ error: "E-Mail bereits vergeben" });
  const user = { id: Date.now(), Name, E-Mail, Rolle: Rolle || "Mitglied", Gruppen-IDs: Gruppen-IDs || [] };
  Benutzer.push(Benutzer);
  res.json(Benutzer);
});

// Benutzer löschen (nur Admin)
app.delete('/api/users/:id', (req, res) => {
  const { requesterRole } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  Benutzer = Benutzer.Filter(u => u.id !== Nummer(req.params.id));
  res.json({ Erfolg: wahr });
});

// Gruppen einem Benutzer zuweisen (nur Admin)
app.put('/api/users/:id/groups', (req, res) => {
  const { Gruppen-IDs, Requester-Rolle } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  Benutzer = Benutzer.map(u => u.id === Nummer(req.params.id) ? { ...u, Gruppen-IDs } : u);
  res.json({ Erfolg: wahr });
});

// Nutzer-Login (sehr einfach, nur zum Testen!)
app.post('/api/login', (req, res) => {
  const { E-Mail } = req.body;
  const Benutzer = Benutzer.find(u => u.email === E-Mail);
  if (!user) return res.status(404).json({ error: "User nicht gefunden" });
  res.json(Benutzer);
});

/** Gruppen-API **/
// Gruppe anlegen (nur Admin)
app.post('/api/groups', (req, res) => {
  const { name, requesterRole } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  if (!name) return res.status(400).json({ error: "Gruppenname erforderlich" });
  const Gruppe = { ID: Date.now(), Name };
  groups.push(Gruppe);
  res.json(Gruppe);
});

// Gruppe löschen (nur Admin)
app.delete('/api/groups/:id', (req, res) => {
  const { requesterRole } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  Gruppen = Gruppen.Filter(g => g.id !== Nummer(req.params.id));
  // Entferne gelöschte Gruppe aus allen Nutzern
  Benutzer = Benutzer.map(u => ({
    ...du,
    Gruppen-IDs: u.groupIds.filter(id => id !== Nummer(req.params.id))
  }));
  res.json({ Erfolg: wahr });
});

/** Alarm-API **/
// Alarm anlegen (nur Admin)
app.post('/api/alarms', (req, res) => {
  const { Titel, Nachricht, Gruppen-IDs, Sounddatei, Standort, Anfordererrolle } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  wenn (!Titel || !Nachricht || !Array.isArray(groupIds) || groupIds.length === 0) {
    return res.status(400).json({ error: "Titel, Nachricht und mindestens eine Gruppe erforderlich" });
  }
  const alarm = { id: Date.now(), Titel, Nachricht, Gruppen-IDs, Sounddatei, Ort, Zeit: neues Datum() };
  alarme.push(alarm);
  res.json(alarm);
});

// Rückmeldung auf Alarm (nur Gruppenmitglied)
app.post('/api/alarms/:id/response', (req, res) => {
  const { Benutzer-ID, Antwort } = req.body;
  if (!userId) return res.status(400).json({ error: "userId erforderlich" });
  // Prüfen, ob User Teil der Alarmgruppe ist
  const alarm = alarms.find(a => a.id === Nummer(req.params.id));
  const Benutzer = Benutzer.find(u => u.id === Benutzer-ID);
  wenn (!alarm || !benutzer || !benutzer.groupIds.some(gid => alarm.groupIds.includes(gid))) {
    return res.status(403).json({ error: "Keine Berechtigung" });
  }
  Antworten = Antworten.filter(r => !(r.alarmId === alarm.id && r.userId === userId)); // nur eine Antwort pro User/Alarm
  Antworten.push({
    alarmId: Nummer(req.params.id),
    Benutzer-ID,
    Antwort,
    Zeit: neues Datum()
  });
  res.json({ Erfolg: wahr });
});

// Rückmeldungen zu Alarm (nur Admin)
app.get('/api/alarms/:id/responses', (req, res) => {
  const { requesterRole } = req.query;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  res.json(Antworten.Filter(r => r.AlarmId === Nummer(req.params.id)));
});

/** Sound-Upload (nur Admin) **/
app.post('/api/sounds', upload.single('sound'), (req, res) => {
  const { requesterRole } = req.body;
  if (requesterRole !== "admin") return res.status(403).json({ error: "Keine Berechtigung" });
  if (!req.file) return res.status(400).json({ error: "Keine Datei erhalten" });
  const Ton = {
    ID: Date.now(),
    Dateiname: req.file.filename,
    Originalname: req.file.originalname
  };
  sounds.push(Ton);
  res.json(Ton);
});

/** Listen-APIs **/
app.get('/api/groups', (req,res) => res.json(groups));
app.get('/api/users', (req,res) => res.json(Benutzer));
app.get('/api/alarms', (req,res) => res.json(alarms));
app.get('/api/sounds', (req,res) => res.json(sounds));
app.use('/uploads', express.static('uploads'));

// Gesundheitscheck
app.get('/', (req, res) => res.send('BS-ALARM API läuft!'));

app.listen(PORT, () => {
  console.log(`BS-ALARM-API läuft auf http://localhost:${PORT}`);
});
