import * as satellite from "https://cdn.jsdelivr.net/npm/satellite.js@7.0.1/+esm";

const LOCAL_DATA_URL = "./data/starlink.json";

const STAR_CATALOG = {
  "베가 (Vega)": { raHours: 18.615649, decDeg: 38.783689 },
  "데네브 (Deneb)": { raHours: 20.690532, decDeg: 45.280338 },
  "알타이르 (Altair)": { raHours: 19.846389, decDeg: 8.868333 },
  "북극성 (Polaris)": { raHours: 2.530301, decDeg: 89.264109 },
  "시리우스 (Sirius)": { raHours: 6.752481, decDeg: -16.716116 },
  "베텔게우스 (Betelgeuse)": { raHours: 5.919529, decDeg: 7.407064 },
};

const EARTH_RADIUS_KM = 6378.137;
let ommData = [];
let lastEvents = [];
let lastSchedule = [];

const $ = (id) => document.getElementById(id);

const elements = {
  loadLatestButton: $("loadLatestButton"),
  ommFile: $("ommFile"),
  downloadDataButton: $("downloadDataButton"),
  analyzeButton: $("analyzeButton"),
  dataStatus: $("dataStatus"),
  dataDetail: $("dataDetail"),
  dataStatusDot: $("dataStatusDot"),
  dateInput: $("dateInput"),
  targetSelect: $("targetSelect"),
  progressBox: $("progressBox"),
  progressBar: $("progressBar"),
  progressText: $("progressText"),
  resultsSection: $("resultsSection"),
  scheduleBody: $("scheduleBody"),
  eventsBody: $("eventsBody"),
  riskChart: $("riskChart"),
};

initialize();

function initialize() {
  Object.keys(STAR_CATALOG).forEach((name) => {
    const option = document.createElement("option");
    option.value = name;
    option.textContent = name;
    elements.targetSelect.appendChild(option);
  });

  const nowKst = new Date(
    new Date().toLocaleString("en-US", { timeZone: "Asia/Seoul" }),
  );
  elements.dateInput.value = [
    nowKst.getFullYear(),
    String(nowKst.getMonth() + 1).padStart(2, "0"),
    String(nowKst.getDate()).padStart(2, "0"),
  ].join("-");

  elements.loadLatestButton.addEventListener("click", loadLatestData);
  elements.ommFile.addEventListener("change", loadUploadedFile);
  elements.downloadDataButton.addEventListener("click", () => {
    downloadJson("starlink_omm_data.json", ommData);
  });
  elements.analyzeButton.addEventListener("click", analyze);
  $("downloadScheduleButton").addEventListener("click", () => {
    downloadCsv("optimized_schedule.csv", lastSchedule);
  });
  $("downloadEventsButton").addEventListener("click", () => {
    downloadCsv("satellite_risk_events.csv", lastEvents);
  });

  // GitHub Actions가 저장한 동일 출처 JSON을 페이지 시작 시 자동으로 읽습니다.
  window.setTimeout(loadLatestData, 250);
}

async function loadLatestData() {
  setLoadingStatus(
    "저장소 자료 확인 중",
    "GitHub Actions가 갱신한 Starlink OMM 자료를 불러오고 있습니다.",
  );

  try {
    const response = await fetch(`${LOCAL_DATA_URL}?v=${Date.now()}`, {
      cache: "no-store",
      headers: { Accept: "application/json" },
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const data = await response.json();
    validateOmmData(data);
    setOmmData(data, "GitHub 자동 갱신 자료");
  } catch (error) {
    setErrorStatus(
      "자동 자료가 아직 없습니다",
      "GitHub Actions에서 'Update satellite data'를 한 번 실행한 뒤 페이지를 새로고침해 주세요.",
    );
    console.error(error);
  }
}

async function loadUploadedFile(event) {
  const file = event.target.files?.[0];
  if (!file) return;

  setLoadingStatus("파일 읽는 중", file.name);

  try {
    const text = await file.text();
    const data = JSON.parse(text);
    validateOmmData(data);
    setOmmData(data, file.name);
  } catch (error) {
    setErrorStatus("파일 형식 오류", "CelesTrak OMM JSON 배열인지 확인해 주세요.");
    console.error(error);
  }
}

function validateOmmData(data) {
  if (!Array.isArray(data) || data.length === 0) {
    throw new Error("OMM 데이터가 배열이 아니거나 비어 있습니다.");
  }

  const sample = data[0];
  const required = [
    "OBJECT_NAME",
    "EPOCH",
    "MEAN_MOTION",
    "ECCENTRICITY",
    "INCLINATION",
    "RA_OF_ASC_NODE",
    "ARG_OF_PERICENTER",
    "MEAN_ANOMALY",
    "NORAD_CAT_ID",
  ];

  const missing = required.filter((key) => !(key in sample));
  if (missing.length > 0) {
    throw new Error(`필수 필드 누락: ${missing.join(", ")}`);
  }
}

function setOmmData(data, sourceName) {
  ommData = data;
  elements.dataStatus.textContent = "위성 데이터 준비 완료";
  elements.dataDetail.textContent =
    `${sourceName} · ${data.length.toLocaleString()}개 객체`;
  elements.dataStatusDot.classList.add("ready");
  elements.dataDetail.classList.remove("error");
  elements.analyzeButton.disabled = false;
  elements.downloadDataButton.disabled = false;
}

function setLoadingStatus(title, detail) {
  elements.dataStatus.textContent = title;
  elements.dataDetail.textContent = detail;
  elements.dataStatusDot.classList.remove("ready");
  elements.dataDetail.classList.remove("error");
}

function setErrorStatus(title, detail) {
  elements.dataStatus.textContent = title;
  elements.dataDetail.textContent = detail;
  elements.dataStatusDot.classList.remove("ready");
  elements.dataDetail.classList.add("error");
}

async function analyze() {
  if (!ommData.length) {
    alert("먼저 위성 데이터를 불러와 주세요.");
    return;
  }

  try {
    const settings = readSettings();
    validateSettings(settings);

    elements.analyzeButton.disabled = true;
    elements.resultsSection.classList.add("hidden");
    showProgress(0, "위성 자료를 준비하는 중입니다.");

    const selected = selectSatellites(ommData, settings.satelliteLimit, 42);
    const satrecs = selected.map((omm) => ({
      name: omm.OBJECT_NAME,
      noradId: Number(omm.NORAD_CAT_ID),
      epoch: new Date(`${omm.EPOCH}Z`),
      satrec: satellite.json2satrec(omm),
    }));

    const times = buildTimeGrid(settings.startDate, settings.endDate, settings.stepSeconds);
    const target = STAR_CATALOG[settings.targetName];

    const events = [];

    for (let satIndex = 0; satIndex < satrecs.length; satIndex += 1) {
      const item = satrecs[satIndex];
      const samples = [];

      for (let timeIndex = 0; timeIndex < times.length; timeIndex += 1) {
        const date = times[timeIndex];
        const pv = satellite.propagate(item.satrec, date);
        if (!pv?.position) continue;

        const gmst = satellite.gstime(date);
        const positionEcf = satellite.eciToEcf(pv.position, gmst);
        const observerGd = {
          longitude: toRadians(settings.longitude),
          latitude: toRadians(settings.latitude),
          height: settings.heightM / 1000,
        };

        const look = satellite.ecfToLookAngles(observerGd, positionEcf);
        const satAlt = look.elevation;
        const satAz = normalizeRadians(look.azimuth);

        const targetHorizontal = equatorialToHorizontal(
          target.raHours,
          target.decDeg,
          settings.latitude,
          settings.longitude,
          date,
        );

        const sun = solarEquatorial(date);
        const sunHorizontal = equatorialToHorizontal(
          sun.raHours,
          sun.decDeg,
          settings.latitude,
          settings.longitude,
          date,
        );

        const separation = angularSeparationHorizontal(
          satAlt,
          satAz,
          targetHorizontal.altitude,
          targetHorizontal.azimuth,
        );

        const sunlit = isSatelliteSunlitCylindrical(pv.position, sun);

        const usable =
          toDegrees(targetHorizontal.altitude) >= settings.targetMinAltitude &&
          toDegrees(sunHorizontal.altitude) <= settings.sunAltitudeLimit;

        const risky =
          usable &&
          toDegrees(satAlt) >= settings.satelliteMinAltitude &&
          sunlit &&
          toDegrees(separation) <= settings.riskRadius;

        samples.push({
          time: date,
          risky,
          separationDeg: toDegrees(separation),
          altitudeDeg: toDegrees(satAlt),
          rangeKm: look.rangeSat,
        });
      }

      events.push(...groupRiskEvents(item, samples, settings.stepSeconds));

      if (satIndex % 5 === 0 || satIndex === satrecs.length - 1) {
        const progress = ((satIndex + 1) / satrecs.length) * 100;
        showProgress(
          progress,
          `위성 ${satIndex + 1}/${satrecs.length}개 계산 완료`,
        );
        await allowBrowserPaint();
      }
    }

    events.sort((a, b) => a.startDate - b.startDate);

    const baselineTimes = buildBaselineTimes(
      settings.startDate,
      settings.endDate,
      settings.exposureSeconds,
      settings.intervalMinutes,
    );

    const schedule = optimizeSchedule(
      events,
      baselineTimes,
      settings.startDate,
      new Date(settings.endDate.getTime() - settings.exposureSeconds * 1000),
      settings.exposureSeconds,
      settings.shiftMinutes * 60,
    );

    lastEvents = events.map(eventToExportRow);
    lastSchedule = schedule.map(scheduleToExportRow);

    renderResults(satrecs.length, events, schedule);
    showProgress(100, "분석이 완료되었습니다.");
    setTimeout(() => elements.progressBox.classList.add("hidden"), 500);
  } catch (error) {
    console.error(error);
    alert(`분석을 실행하지 못했습니다.\n${error.message}`);
    elements.progressBox.classList.add("hidden");
  } finally {
    elements.analyzeButton.disabled = false;
  }
}

function readSettings() {
  const date = $("dateInput").value;
  const start = $("startTime").value;
  const end = $("endTime").value;

  return {
    startDate: parseKstDateTime(date, start),
    endDate: parseKstDateTime(date, end),
    targetName: $("targetSelect").value,
    latitude: Number($("latitude").value),
    longitude: Number($("longitude").value),
    heightM: Number($("height").value),
    satelliteLimit: Number($("satelliteLimit").value),
    stepSeconds: Number($("stepSeconds").value),
    fovDiameter: Number($("fovDiameter").value),
    exposureSeconds: Number($("exposureSeconds").value),
    intervalMinutes: Number($("intervalMinutes").value),
    shiftMinutes: Number($("shiftMinutes").value),
    targetMinAltitude: Number($("targetMinAltitude").value),
    satelliteMinAltitude: Number($("satelliteMinAltitude").value),
    sunAltitudeLimit: Number($("sunAltitudeLimit").value),
    safetyMargin: Number($("safetyMargin").value),
    get riskRadius() {
      return this.fovDiameter / 2 + this.safetyMargin;
    },
  };
}

function validateSettings(settings) {
  if (!(settings.endDate > settings.startDate)) {
    throw new Error("종료 시각은 시작 시각보다 늦어야 합니다.");
  }

  const durationHours =
    (settings.endDate - settings.startDate) / (1000 * 60 * 60);

  if (durationHours > 4) {
    throw new Error("브라우저 계산량을 고려하여 분석 시간은 4시간 이하로 설정해 주세요.");
  }

  if (settings.satelliteLimit > ommData.length) {
    settings.satelliteLimit = ommData.length;
  }

  const newestEpoch = Math.max(
    ...ommData.slice(0, Math.min(500, ommData.length)).map(
      (row) => new Date(`${row.EPOCH}Z`).getTime(),
    ),
  );
  const epochGapDays = Math.abs(settings.startDate.getTime() - newestEpoch) / 86400000;

  if (epochGapDays > 14) {
    const proceed = confirm(
      `관측 날짜와 궤도 기준일이 약 ${epochGapDays.toFixed(1)}일 차이 납니다.\n` +
      "위치 오차가 커질 수 있습니다. 그래도 계속하시겠습니까?",
    );
    if (!proceed) {
      throw new Error("분석이 취소되었습니다.");
    }
  }
}

function groupRiskEvents(item, samples, stepSeconds) {
  const events = [];
  let startIndex = null;

  const closeEvent = (endIndex) => {
    if (startIndex === null) return;

    const group = samples.slice(startIndex, endIndex + 1);
    let closest = group[0];

    for (const sample of group) {
      if (sample.separationDeg < closest.separationDeg) {
        closest = sample;
      }
    }

    events.push({
      satellite: item.name,
      noradId: item.noradId,
      startDate: group[0].time,
      endDate: new Date(group[group.length - 1].time.getTime() + stepSeconds * 1000),
      closestDate: closest.time,
      minimumSeparationDeg: closest.separationDeg,
      maximumAltitudeDeg: Math.max(...group.map((row) => row.altitudeDeg)),
      minimumDistanceKm: Math.min(...group.map((row) => row.rangeKm)),
    });

    startIndex = null;
  };

  samples.forEach((sample, index) => {
    if (sample.risky && startIndex === null) {
      startIndex = index;
    }

    const nextIsSafe = index === samples.length - 1 || !samples[index + 1].risky;
    if (sample.risky && nextIsSafe) {
      closeEvent(index);
    }
  });

  return events;
}

function optimizeSchedule(
  events,
  baselineTimes,
  observationStart,
  lastStart,
  exposureSeconds,
  maxShiftSeconds,
) {
  const candidateStepSeconds = 30;
  const offsets = [];

  for (
    let offset = -maxShiftSeconds;
    offset <= maxShiftSeconds;
    offset += candidateStepSeconds
  ) {
    offsets.push(offset);
  }

  if (!offsets.includes(0)) offsets.push(0);
  offsets.sort((a, b) => a - b);

  const rows = [];
  let previousEnd = null;

  for (const baseTime of baselineTimes) {
    const candidates = offsets
      .map((offset) => ({
        offset,
        time: new Date(baseTime.getTime() + offset * 1000),
      }))
      .filter(({ time }) => {
        if (time < observationStart || time > lastStart) return false;
        if (previousEnd && time < previousEnd) return false;
        return true;
      })
      .map(({ offset, time }) => ({
        offset,
        time,
        riskCount: exposureRiskCount(events, time, exposureSeconds),
      }))
      .sort(
        (a, b) =>
          a.riskCount - b.riskCount ||
          Math.abs(a.offset) - Math.abs(b.offset),
      );

    const chosen = candidates[0] ?? {
      offset: 0,
      time: baseTime,
      riskCount: exposureRiskCount(events, baseTime, exposureSeconds),
    };

    const baselineRisk = exposureRiskCount(events, baseTime, exposureSeconds);

    rows.push({
      baselineTime: baseTime,
      baselineRisk,
      optimizedTime: chosen.time,
      optimizedRisk: chosen.riskCount,
      shiftSeconds: chosen.offset,
    });

    previousEnd = new Date(chosen.time.getTime() + exposureSeconds * 1000);
  }

  return rows;
}

function exposureRiskCount(events, startTime, exposureSeconds) {
  const endTime = new Date(startTime.getTime() + exposureSeconds * 1000);
  const names = new Set();

  events.forEach((event) => {
    if (event.startDate < endTime && event.endDate >= startTime) {
      names.add(event.satellite);
    }
  });

  return names.size;
}

function renderResults(analyzedSatellites, events, schedule) {
  const baselineRisk = schedule.filter((row) => row.baselineRisk > 0).length;
  const optimizedRisk = schedule.filter((row) => row.optimizedRisk > 0).length;
  const reduction =
    baselineRisk > 0
      ? ((baselineRisk - optimizedRisk) / baselineRisk) * 100
      : 0;

  $("metricSatellites").textContent = analyzedSatellites.toLocaleString();
  $("metricEvents").textContent = events.length.toLocaleString();
  $("metricRiskChange").textContent = `${baselineRisk} → ${optimizedRisk}`;
  $("metricReduction").textContent = `${reduction.toFixed(1)}%`;

  elements.scheduleBody.innerHTML = "";
  schedule.forEach((row) => {
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${formatKst(row.baselineTime)}</td>
      <td>${row.baselineRisk}</td>
      <td>${formatKst(row.optimizedTime)}</td>
      <td>${row.optimizedRisk}</td>
      <td>${row.shiftSeconds}</td>
    `;
    elements.scheduleBody.appendChild(tr);
  });

  elements.eventsBody.innerHTML = "";
  events.slice(0, 500).forEach((event) => {
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${escapeHtml(event.satellite)}</td>
      <td>${event.noradId}</td>
      <td>${formatKst(event.startDate)}</td>
      <td>${formatKst(event.endDate)}</td>
      <td>${event.minimumSeparationDeg.toFixed(3)}</td>
      <td>${event.maximumAltitudeDeg.toFixed(2)}</td>
      <td>${event.minimumDistanceKm.toFixed(1)}</td>
    `;
    elements.eventsBody.appendChild(tr);
  });

  drawRiskChart(elements.riskChart, baselineRisk, optimizedRisk, schedule.length);
  elements.resultsSection.classList.remove("hidden");
  elements.resultsSection.scrollIntoView({ behavior: "smooth", block: "start" });
}

function drawRiskChart(canvas, baselineRisk, optimizedRisk, total) {
  const dpr = window.devicePixelRatio || 1;
  const width = canvas.clientWidth || 900;
  const height = 300;

  canvas.width = width * dpr;
  canvas.height = height * dpr;

  const ctx = canvas.getContext("2d");
  ctx.scale(dpr, dpr);
  ctx.clearRect(0, 0, width, height);

  const values = [baselineRisk, optimizedRisk];
  const labels = ["기존 일정", "최적화 일정"];
  const maxValue = Math.max(total, 1);
  const chartTop = 35;
  const chartBottom = 245;
  const chartHeight = chartBottom - chartTop;
  const barWidth = Math.min(150, width / 5);
  const centers = [width * 0.34, width * 0.66];

  ctx.strokeStyle = "rgba(168,183,203,0.28)";
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.moveTo(45, chartBottom);
  ctx.lineTo(width - 30, chartBottom);
  ctx.stroke();

  values.forEach((value, index) => {
    const barHeight = (value / maxValue) * chartHeight;
    const x = centers[index] - barWidth / 2;
    const y = chartBottom - barHeight;

    const gradient = ctx.createLinearGradient(0, y, 0, chartBottom);
    gradient.addColorStop(0, index === 0 ? "#ff9c82" : "#72d6ff");
    gradient.addColorStop(1, index === 0 ? "#b94545" : "#167eb0");

    ctx.fillStyle = gradient;
    roundRect(ctx, x, y, barWidth, Math.max(barHeight, 3), 12);
    ctx.fill();

    ctx.fillStyle = "#eef5ff";
    ctx.font = "700 20px system-ui";
    ctx.textAlign = "center";
    ctx.fillText(String(value), centers[index], Math.max(y - 10, 25));

    ctx.fillStyle = "#a8b7cb";
    ctx.font = "600 14px system-ui";
    ctx.fillText(labels[index], centers[index], 275);
  });
}

function roundRect(ctx, x, y, width, height, radius) {
  const r = Math.min(radius, width / 2, height / 2);
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + width, y, x + width, y + height, r);
  ctx.arcTo(x + width, y + height, x, y + height, r);
  ctx.arcTo(x, y + height, x, y, r);
  ctx.arcTo(x, y, x + width, y, r);
  ctx.closePath();
}

function equatorialToHorizontal(raHours, decDeg, latDeg, lonDeg, date) {
  const gmst = satellite.gstime(date);
  const ra = toRadians(raHours * 15);
  const dec = toRadians(decDeg);
  const lat = toRadians(latDeg);
  const lst = normalizeRadians(gmst + toRadians(lonDeg));
  const hourAngle = normalizeSignedRadians(lst - ra);

  const sinAlt =
    Math.sin(dec) * Math.sin(lat) +
    Math.cos(dec) * Math.cos(lat) * Math.cos(hourAngle);
  const altitude = Math.asin(clamp(sinAlt, -1, 1));

  const cosAlt = Math.max(Math.cos(altitude), 1e-12);
  const sinAz = (-Math.cos(dec) * Math.sin(hourAngle)) / cosAlt;
  const cosAz =
    (Math.sin(dec) - Math.sin(altitude) * Math.sin(lat)) /
    (cosAlt * Math.cos(lat));

  const azimuth = normalizeRadians(Math.atan2(sinAz, cosAz));
  return { altitude, azimuth };
}

function angularSeparationHorizontal(alt1, az1, alt2, az2) {
  const cosine =
    Math.sin(alt1) * Math.sin(alt2) +
    Math.cos(alt1) * Math.cos(alt2) * Math.cos(az1 - az2);
  return Math.acos(clamp(cosine, -1, 1));
}

function solarEquatorial(date) {
  const jd = julianDate(date);
  const n = jd - 2451545.0;
  const meanLongitude = normalizeDegrees(280.460 + 0.9856474 * n);
  const meanAnomaly = toRadians(normalizeDegrees(357.528 + 0.9856003 * n));
  const eclipticLongitude = toRadians(
    normalizeDegrees(
      meanLongitude +
      1.915 * Math.sin(meanAnomaly) +
      0.020 * Math.sin(2 * meanAnomaly),
    ),
  );
  const obliquity = toRadians(23.439 - 0.0000004 * n);

  const x = Math.cos(eclipticLongitude);
  const y = Math.cos(obliquity) * Math.sin(eclipticLongitude);
  const z = Math.sin(obliquity) * Math.sin(eclipticLongitude);

  const ra = normalizeRadians(Math.atan2(y, x));
  const dec = Math.asin(z);

  return {
    raHours: toDegrees(ra) / 15,
    decDeg: toDegrees(dec),
    unitEci: {
      x: Math.cos(dec) * Math.cos(ra),
      y: Math.cos(dec) * Math.sin(ra),
      z: Math.sin(dec),
    },
  };
}

function isSatelliteSunlitCylindrical(positionEci, sun) {
  const r = positionEci;
  const u = sun.unitEci;
  const dot = r.x * u.x + r.y * u.y + r.z * u.z;

  if (dot >= 0) return true;

  const r2 = r.x * r.x + r.y * r.y + r.z * r.z;
  const perpendicular2 = Math.max(r2 - dot * dot, 0);
  return Math.sqrt(perpendicular2) > EARTH_RADIUS_KM;
}

function julianDate(date) {
  return date.getTime() / 86400000 + 2440587.5;
}

function buildTimeGrid(start, end, stepSeconds) {
  const times = [];
  for (
    let current = start.getTime();
    current < end.getTime();
    current += stepSeconds * 1000
  ) {
    times.push(new Date(current));
  }
  return times;
}

function buildBaselineTimes(start, end, exposureSeconds, intervalMinutes) {
  const result = [];
  const lastStart = end.getTime() - exposureSeconds * 1000;

  for (
    let current = start.getTime();
    current <= lastStart;
    current += intervalMinutes * 60 * 1000
  ) {
    result.push(new Date(current));
  }

  return result;
}

function selectSatellites(data, limit, seed) {
  const copy = [...data];
  let state = seed >>> 0;

  const random = () => {
    state = (1664525 * state + 1013904223) >>> 0;
    return state / 4294967296;
  };

  for (let index = copy.length - 1; index > 0; index -= 1) {
    const swapIndex = Math.floor(random() * (index + 1));
    [copy[index], copy[swapIndex]] = [copy[swapIndex], copy[index]];
  }

  return copy.slice(0, Math.min(limit, copy.length));
}

function parseKstDateTime(dateString, timeString) {
  return new Date(`${dateString}T${timeString}:00+09:00`);
}

function formatKst(date) {
  return new Intl.DateTimeFormat("ko-KR", {
    timeZone: "Asia/Seoul",
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
    hour: "2-digit",
    minute: "2-digit",
    second: "2-digit",
    hour12: false,
  }).format(date);
}

function eventToExportRow(event) {
  return {
    satellite: event.satellite,
    norad_id: event.noradId,
    start_kst: formatKst(event.startDate),
    end_kst: formatKst(event.endDate),
    closest_kst: formatKst(event.closestDate),
    minimum_separation_deg: event.minimumSeparationDeg.toFixed(5),
    maximum_altitude_deg: event.maximumAltitudeDeg.toFixed(3),
    minimum_distance_km: event.minimumDistanceKm.toFixed(3),
  };
}

function scheduleToExportRow(row) {
  return {
    baseline_time_kst: formatKst(row.baselineTime),
    baseline_risk_satellites: row.baselineRisk,
    optimized_time_kst: formatKst(row.optimizedTime),
    optimized_risk_satellites: row.optimizedRisk,
    time_shift_seconds: row.shiftSeconds,
  };
}

function downloadJson(filename, value) {
  const blob = new Blob([JSON.stringify(value, null, 2)], {
    type: "application/json;charset=utf-8",
  });
  downloadBlob(filename, blob);
}

function downloadCsv(filename, rows) {
  if (!rows.length) {
    alert("저장할 결과가 없습니다.");
    return;
  }

  const headers = Object.keys(rows[0]);
  const escape = (value) => {
    const text = String(value ?? "");
    return `"${text.replaceAll('"', '""')}"`;
  };

  const csv = [
    headers.map(escape).join(","),
    ...rows.map((row) => headers.map((key) => escape(row[key])).join(",")),
  ].join("\r\n");

  const blob = new Blob(["\ufeff", csv], {
    type: "text/csv;charset=utf-8",
  });
  downloadBlob(filename, blob);
}

function downloadBlob(filename, blob) {
  const url = URL.createObjectURL(blob);
  const anchor = document.createElement("a");
  anchor.href = url;
  anchor.download = filename;
  document.body.appendChild(anchor);
  anchor.click();
  anchor.remove();
  URL.revokeObjectURL(url);
}

function showProgress(percent, text) {
  elements.progressBox.classList.remove("hidden");
  elements.progressBar.style.width = `${clamp(percent, 0, 100)}%`;
  elements.progressText.textContent = text;
}

function allowBrowserPaint() {
  return new Promise((resolve) => requestAnimationFrame(resolve));
}

function toRadians(degrees) {
  return (degrees * Math.PI) / 180;
}

function toDegrees(radians) {
  return (radians * 180) / Math.PI;
}

function normalizeRadians(value) {
  const twoPi = Math.PI * 2;
  return ((value % twoPi) + twoPi) % twoPi;
}

function normalizeSignedRadians(value) {
  const normalized = normalizeRadians(value);
  return normalized > Math.PI ? normalized - Math.PI * 2 : normalized;
}

function normalizeDegrees(value) {
  return ((value % 360) + 360) % 360;
}

function clamp(value, minimum, maximum) {
  return Math.min(Math.max(value, minimum), maximum);
}

function escapeHtml(value) {
  return String(value)
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#039;");
}
