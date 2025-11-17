<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Aplikasi Kalkulator Teknik Lingkungan - Web Version</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; padding: 0; background: #f2f2f2; }
    header { background: #2d6cdf; color: white; padding: 18px; text-align: center; font-size: 22px; }
    .container { max-width: 1000px; margin: 24px auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 6px 18px rgba(0,0,0,0.08); }
    h2 { margin-top: 0; }
    .section { margin-top: 18px; padding-bottom: 18px; border-bottom: 1px solid #eee; }
    label { font-weight: 600; display:block; margin-top:8px; }
    input, select { padding:8px; width:100%; margin-top:6px; border-radius:6px; border:1px solid #cfcfcf; box-sizing:border-box; }
    .row { display:flex; gap:12px; }
    .col { flex:1; }
    .result { background:#eef6ff; padding:12px; border-radius:8px; margin-top:12px; font-size:15px; }
    .hint { font-size:13px; color:#555; margin-top:8px; }
    button { background:#2d6cdf; color:white; border:none; padding:10px 14px; border-radius:8px; cursor:pointer; margin-top:10px; }
    button:hover{ opacity:0.95 }
    .small { font-size:13px; color:#444 }
    .flex { display:flex; gap:12px; align-items:center; }
    @media(max-width:760px){ .row{flex-direction:column} }
  </style>
</head>
<body>
  <header>🌍 Kalkulator Teknik Lingkungan – Versi Web (dengan Modul Sedimentasi)</header>

  <div class="container">
    <h2>1. Kalkulator Volume Reaktor (HRT)</h2>
    <div class="section">
      <div class="row">
        <div class="col">
          <label>Debit Q (m³/hari)</label>
          <input type="number" id="q_m3day" value="500" />
        </div>
        <div class="col">
          <label>HRT (jam)</label>
          <input type="number" id="hrt_hours" value="24" />
        </div>
      </div>
      <button onclick="hitungHRT()">Hitung Volume</button>
      <div id="hasilHRT" class="result"></div>
    </div>

    <h2>2. Konversi Debit</h2>
    <div class="section">
      <div class="row">
        <div class="col">
          <label>Nilai Input</label>
          <input type="number" id="conv_in" value="100" />
        </div>
        <div class="col">
          <label>Dari</label>
          <select id="conv_from">
            <option value="m3day">m³/hari</option>
            <option value="ls">L/detik</option>
          </select>
        </div>
        <div class="col">
          <label>Ke</label>
          <select id="conv_to">
            <option value="ls">L/detik</option>
            <option value="m3day">m³/hari</option>
          </select>
        </div>
      </div>
      <button onclick="konversiDebit()">Konversi</button>
      <div id="hasilKonversi" class="result"></div>
    </div>

    <h2>3. Modul Sedimentasi (Lengkap)</h2>
    <div class="section">
      <p class="hint">Modul ini menghitung dimensi kolam pengendap berdasarkan <b>Detention Time</b>, <b>Surface Overflow Rate (SOR)</b>, dan <b>Volume</b>. Juga menampilkan klasifikasi SOR dan rekomendasi.</p>

      <div class="row">
        <div class="col">
          <label>Debit masuk Q (m³/hari)</label>
          <input type="number" id="s_q" value="300" />

          <label>Waktu detensi yang diinginkan (jam)</label>
          <input type="number" id="s_detention_hours" value="6" step="0.5" />

          <label>Kedalaman operasional (m)</label>
          <input type="number" id="s_depth" value="2.0" step="0.1" />

          <label>Freeboard / ruang atas (m)</label>
          <input type="number" id="s_freeboard" value="0.5" step="0.1" />
        </div>

        <div class="col">
          <label>Lebar kolam (m) — untuk estimasi panjang</label>
          <input type="number" id="s_width" value="6.0" step="0.1" />

          <label>Ingin mengatur SOR (Surface Overflow Rate)?</label>
          <div class="flex">
            <input type="number" id="s_sor_value" value="30" step="0.1" style="width:120px" />
            <select id="s_sor_unit">
              <option value="m3_m2_day">m³/m²·hari</option>
              <option value="m_day">m/day</option>
            </select>
          </div>

          <label>Atau masukkan panjang yang diinginkan (m) — kosongkan atau 0 untuk abaikan</label>
          <input type="number" id="s_length_override" value="0" step="0.1" />

          <label>Hitung efisiensi pengendapan untuk partikel dengan kecepatan te (mm/s)</label>
          <input type="number" id="s_settling_v_mm_s" value="0.5" step="0.01" />
          <div class="small">(Estimasi kasar: jika v > v0 maka partikel bisa tersisih)</div>
        </div>
      </div>

      <button onclick="hitungSedimentasi()">Hitung Dimensi Sedimentasi</button>

      <div id="hasilSed" class="result"></div>

      <div id="notes" class="hint"></div>
    </div>

    <div style="margin-top:18px; font-size:13px; color:#333">Catatan: rumus yang digunakan adalah estimasi desain dasar. Untuk desain akhir selalu cek standar lokal (SNI) atau panduan teknis yang relevan.</div>
  </div>

  <script>
    function hitungHRT(){
      const Q = parseFloat(document.getElementById("q_m3day").value) || 0;
      const HRT = parseFloat(document.getElementById("hrt_hours").value) || 0;
      const HRT_days = HRT / 24;
      const volume = Q * HRT_days;
      document.getElementById("hasilHRT").innerHTML = `Volume Reaktor: <b>${volume.toFixed(3)} m³</b> (HRT ${HRT} jam)`;
    }

    function konversiDebit(){
      const val = parseFloat(document.getElementById("conv_in").value) || 0;
      const from = document.getElementById("conv_from").value;
      const to = document.getElementById("conv_to").value;
      let hasil = 0;
      if(from===to) hasil=val;
      else if(from==='m3day' && to==='ls') hasil = val*1000/86400;
      else if(from==='ls' && to==='m3day') hasil = val*86400/1000;
      document.getElementById("hasilKonversi").innerHTML = `Hasil: <b>${hasil.toFixed(6)}</b> ${to}`;
    }

    function hitungSedimentasi(){
      const Q = parseFloat(document.getElementById('s_q').value) || 0; // m3/day
      const detention_h = parseFloat(document.getElementById('s_detention_hours').value) || 0; // hours
      const depth = parseFloat(document.getElementById('s_depth').value) || 0; // m
      const freeboard = parseFloat(document.getElementById('s_freeboard').value) || 0; // m
      const width = parseFloat(document.getElementById('s_width').value) || 0; // m
      const sor_val = parseFloat(document.getElementById('s_sor_value').value) || 0; // value
      const sor_unit = document.getElementById('s_sor_unit').value;
      const length_override = parseFloat(document.getElementById('s_length_override').value) || 0; // m
      const sett_v_mm_s = parseFloat(document.getElementById('s_settling_v_mm_s').value) || 0; // mm/s

      // 1) Volume berdasarkan detention time
      const volume = Q * (detention_h/24.0); // m3

      // 2) Surface overflow rate (v0)
      // If user entered in m3/m2.day -> area = Q / SOR
      // If user entered in m/day -> interpret as depth flow velocity (m/day) => area = Q / v (same units)
      let area = 0;
      if(sor_unit === 'm3_m2_day'){
        if(sor_val>0) area = Q / sor_val; // m2
      } else {
        // m/day provided: same as volume/day divided by depth-> unclear; interpret as v (m/day), so area = Q / v
        if(sor_val>0) area = Q / sor_val;
      }

      // 3) If area computed, compute depth and length
      const operative_depth = Math.max(0.2, depth - freeboard); // avoid zero
      let length = 0;
      if(area>0 && width>0){
        length = area / width;
      }

      // If user overrides length, recalc area
      if(length_override > 0){
        length = length_override;
        area = length * width;
      }

      // 4) Recompute actual overflow rate v0 (m3/m2.day)
      let v0 = 0;
      if(area>0) v0 = Q / area; // m3/m2.day

      // 5) depth check from volume
      let depth_from_volume = 0;
      if(area>0) depth_from_volume = volume / area; // m

      // 6) settling velocity comparison: convert mm/s to m/day for comparison with v0
      // v_settling (m/day) = mm/s * 1e-3 (m/mm) * 86400 (s/day)
      const sett_v_m_day = sett_v_mm_s * 1e-3 * 86400;

      // 7) Estimate fraction of particles removed (very rough): if sett_v_m_day >= v0 => high removal
      let removal_comment = '';
      if(sett_v_mm_s <= 0) removal_comment = 'Tidak ada kecepatan endapan yang dimasukkan.';
      else if(sett_v_m_day >= v0) removal_comment = 'Partikel dengan kecepatan endapan ini kemungkinan besar akan terendapkan (efisiensi tinggi).';
      else removal_comment = 'Partikel ini cenderung tidak terendapkan sempurna (efisiensi rendah) — pertimbangkan peningkatan SOR atau unit prapengendapan.';

      // 8) SOR classification (umum):
      // < 10 m3/m2.day = very low (very fine particles) ; 10-30 moderate ; >30 high
      let sor_class = '';
      if(v0 === 0) sor_class = 'Tidak dapat dihitung (area=0).';
      else if(v0 < 10) sor_class = 'Sangat rendah — cocok untuk partikel sangat halus.';
      else if(v0 < 30) sor_class = 'Sedang — desain umum untuk pengendapan efisien.';
      else sor_class = 'Tinggi — cocok untuk partikel kasar, pengendapan cepat.';

      // 9) Output
      const out = [];
      out.push(`<b>Ringkasan Perhitungan</b>`);
      out.push(`Debit Q = <b>${Q.toFixed(3)}</b> m³/hari`);
      out.push(`Waktu detensi = <b>${detention_h.toFixed(2)}</b> jam → Volume kolam ≈ <b>${volume.toFixed(3)}</b> m³`);
      if(area>0) out.push(`Luas permukaan (area) = <b>${area.toFixed(3)}</b> m² (berdasarkan SOR input)`);
      if(width>0) out.push(`Lebar yang dipakai = <b>${width.toFixed(2)}</b> m → Panjang kolam ≈ <b>${length.toFixed(2)}</b> m`);
      out.push(`Kedalaman operasional (tanpa freeboard) = <b>${operative_depth.toFixed(2)}</b> m`);
      if(area>0) out.push(`Kedalaman yang dihitung dari volume/area = <b>${depth_from_volume.toFixed(3)}</b> m`);
      if(area>0) out.push(`Surface overflow rate (v0) = <b>${v0.toFixed(3)}</b> m³/m²·hari`);
      out.push(`<b>Kecepatan endap (masuk) = ${sett_v_mm_s.toFixed(3)} mm/s → ${sett_v_m_day.toFixed(2)} m/day</b>`);
      out.push(`${removal_comment}`);
      out.push(`<i>Klasifikasi SOR saat ini: ${sor_class}</i>`);

      document.getElementById('hasilSed').innerHTML = out.join('<br>');

      // notes / tips
      const tips = [];
      tips.push('<b>Tips desain singkat:</b>');
      tips.push('• Periksa SOR yang direkomendasikan berdasarkan jenis partikel: partikel halus butuh SOR lebih rendah (<10-20).');
      tips.push('• Pastikan panjang vs lebar tidak terlalu kecil (L/W biasanya 2–6 tergantung desain).');
      tips.push('• Periksa kedalaman operasional dan freeboard sesuai praktik lokal.');
      tips.push('• Untuk perhitungan efisiensi endapan yang lebih akurat, gunakan model hidraulika dan uji sedimentasi lab (Jar test).');

      document.getElementById('notes').innerHTML = tips.join('<br>');
    }
  </script>

</body>
</html>
