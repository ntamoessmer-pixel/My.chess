<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>MP3-Downloader</title>
<style>
  :root { --bg:#ffffff; --fg:#1b1b1f; --card:#f3f4f6; --acc:#2563eb; --bd:#d1d5db; }
  @media (prefers-color-scheme: dark) {
    :root { --bg:#14151a; --fg:#e8e8ec; --card:#1f2128; --acc:#60a5fa; --bd:#363944; }
  }
  body { margin:0; background:var(--bg); color:var(--fg); font:16px/1.5 system-ui, sans-serif; }
  main { max-width:680px; margin:0 auto; padding:24px 16px 48px; }
  h1 { font-size:1.6rem; margin:0 0 4px; }
  p.sub { margin:0 0 20px; opacity:.75; }
  .card { background:var(--card); border:1px solid var(--bd); border-radius:12px; padding:16px; margin-bottom:16px; }
  textarea, select { width:100%; box-sizing:border-box; padding:10px; border-radius:8px;
    border:1px solid var(--bd); background:var(--bg); color:var(--fg); font:inherit; }
  textarea { min-height:140px; resize:vertical; }
  label { display:block; font-weight:600; margin:12px 0 6px; }
  button { background:var(--acc); color:#fff; border:0; border-radius:8px; padding:10px 16px;
    font:inherit; font-weight:600; cursor:pointer; margin:12px 8px 0 0; }
  button.sec { background:transparent; color:var(--fg); border:1px solid var(--bd); }
  pre { background:var(--bg); border:1px solid var(--bd); border-radius:8px; padding:10px;
    overflow-x:auto; font-size:.85rem; white-space:pre-wrap; word-break:break-all; }
  ol { padding-left:20px; } code { background:var(--bg); padding:1px 5px; border-radius:4px; }
</style>
</head>
<body>
<main>
  <h1>MP3-Downloader</h1>
  <p class="sub">Erzeugt eine Windows-Datei, die deine Titel per yt-dlp als MP3 herunterlädt.</p>

  <div class="card">
    <label for="eintraege">Liedtitel oder YouTube-Links (eine Zeile pro Eintrag)</label>
    <textarea id="eintraege" placeholder="Daft Punk Get Lucky&#10;https://www.youtube.com/watch?v=..."></textarea>

    <label for="q">Qualität</label>
    <select id="q">
      <option value="0">Beste</option>
      <option value="5">Mittel</option>
      <option value="9">Klein (niedrige Qualität)</option>
    </select>

    <button id="btnBat">.bat-Datei herunterladen</button>
    <button id="btnCopy" class="sec">Befehle anzeigen</button>
    <pre id="out" hidden></pre>
  </div>

  <div class="card">
    <strong>Einmalige Einrichtung (Windows)</strong>
    <ol>
      <li>Eingabeaufforderung öffnen und <code>winget install yt-dlp</code> eingeben.</li>
      <li>Danach <code>winget install Gyan.FFmpeg</code> eingeben.</li>
      <li>Windows neu starten oder die Eingabeaufforderung neu öffnen.</li>
      <li>Die heruntergeladene .bat-Datei doppelklicken. Die MP3s landen in <code>Musik\MP3-Downloads</code>.</li>
    </ol>
    <small>Bitte nur Inhalte herunterladen, für die du die Rechte hast (eigene Uploads, Creative Commons usw.).</small>
  </div>
</main>

<script>
  const $ = id => document.getElementById(id);

  function zeilen() {
    return $("eintraege").value.split(/\r?\n/).map(s => s.trim().replace(/"/g, "")).filter(Boolean);
  }

  function befehl(eintrag, q, fuerBat) {
    const quelle = /^https?:\/\//i.test(eintrag) ? eintrag : "ytsearch1:" + eintrag;
    const p = fuerBat ? "%%" : "%";
    const ziel = '%USERPROFILE%\\Music\\MP3-Downloads\\' + p + '(title)s.' + p + '(ext)s';
    const src = fuerBat ? quelle.replace(/%/g, "%%") : quelle;
    return 'yt-dlp -x --audio-format mp3 --audio-quality ' + q +
           ' --embed-metadata --embed-thumbnail --no-playlist -o "' + ziel + '" "' + src + '"';
  }

  $("btnBat").onclick = () => {
    const z = zeilen();
    if (!z.length) { alert("Bitte mindestens einen Titel oder Link eingeben."); return; }
    const q = $("q").value;
    const bat = ["@echo off", "chcp 65001 >nul",
                 'mkdir "%USERPROFILE%\\Music\\MP3-Downloads" 2>nul',
                 ...z.map(e => befehl(e, q, true)),
                 "echo.", "echo Fertig. Die Dateien liegen in Musik\\MP3-Downloads.",
                 'start "" "%USERPROFILE%\\Music\\MP3-Downloads"', "pause"].join("\r\n");
    const blob = new Blob([bat], { type: "text/plain;charset=utf-8" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "mp3-download.bat";
    a.click();
    URL.revokeObjectURL(a.href);
  };

  $("btnCopy").onclick = () => {
    const z = zeilen();
    const out = $("out");
    out.hidden = false;
    out.textContent = z.length
      ? z.map(e => befehl(e, $("q").value, false)).join("\n")
      : "Bitte zuerst Titel oder Links eingeben.";
  };
</script>
</body>
</html>
