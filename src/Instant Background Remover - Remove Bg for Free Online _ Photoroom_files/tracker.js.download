/* ==============================================
   Conditional Snitcher Loading v2.0
   - Number SNID: Load legacy Snitcher
   - String SNID: Load new Radar/Snitcher
   - No SNID: Skip Snitcher loading
   ============================================== */
!(function () {
  const node =
    document.currentScript ||
    [...document.querySelectorAll('script[src*="tracker.js"]')].pop();

  if (!node) return;

  const snid = node.dataset.snid;

  // If no SNID provided, skip Snitcher loading entirely
  if (!snid) return;

  const isNumericSnid = /^\d+$/.test(snid);

  if (isNumericSnid) {
    // Legacy Snitcher loading (for numeric SNIDs)
    (function (s, n, i, t, c, h) {
      s.SnitchObject = i;
      s[i] ||
        (s[i] = function () {
          (s[i].q = s[i].q || []).push(arguments);
        });
      s[i].l = +new Date();
      c = n.createElement(t);
      h = n.getElementsByTagName(t)[0];
      c.src = `//snid.snitcher.com/${snid}.js`;
      h.parentNode.insertBefore(c, h);
    })(window, document, "snid", "script");
  } else {
    // New Radar/Snitcher loading (for string SNIDs)
    (function (e) {
      "use strict";
      var t = e && e.namespace;
      if (t && e.profileId && e.cdn) {
        var i = window[t];
        if (i && Array.isArray(i) || (i = window[t] = []), !i.initialized && !i._loaded)
          if (i._loaded) console && console.warn("[Radar] Duplicate initialization attempted");
          else {
            i._loaded = !0;
            [
              "track",
              "page",
              "identify",
              "group",
              "alias",
              "ready",
              "debug",
              "on",
              "off",
              "once",
              "trackClick",
              "trackSubmit",
              "trackLink",
              "trackForm",
              "pageview",
              "screen",
              "reset",
              "register",
              "setAnonymousId",
              "addSourceMiddleware",
              "addIntegrationMiddleware",
              "addDestinationMiddleware",
              "giveCookieConsent",
            ].forEach(function (e) {
              var a;
              i[e] =
                ((a = e),
                function () {
                  var e = window[t];
                  if (e.initialized) return e[a].apply(e, arguments);
                  var i = [].slice.call(arguments);
                  return i.unshift(a), e.push(i), e;
                });
            }),
              -1 === e.apiEndpoint.indexOf("http") &&
                (e.apiEndpoint = "https://" + e.apiEndpoint),
              (i.bootstrap = function () {
                var t,
                  i = document.createElement("script");
                i.async = !0;
                i.type = "text/javascript";
                i.id = "__radar__";
                i.setAttribute("data-settings", JSON.stringify(e));
                i.src = [
                  -1 !== (t = e.cdn).indexOf("http") ? "" : "https://",
                  t,
                  "/releases/latest/radar.min.js",
                ].join("");
                var a = document.scripts[0];
                a.parentNode.insertBefore(i, a);
              }),
              i.bootstrap();
          }
      } else
        "undefined" != typeof console &&
          console.error("[Radar] Configuration incomplete");
    })({
      apiEndpoint: "radar.snitcher.com",
      cdn: "cdn.snitcher.com",
      namespace: "Snitcher",
      profileId: snid,
    });
  }
})();
(() => {
  /* ---- 1. Locate <script> ---- */
  const node =
    document.currentScript ||
    [...document.querySelectorAll('script[src*="tracker.js"]')].pop();
  const BASE = new URL(".", node.src);
  const COLLECT = new URL("collect", BASE).href;

  if (!node) return console.error("rfrly: tag not found");

  const SNID = node.dataset.snid;
  const RFID = node.dataset.rfid;
  if (!RFID)
    return console.error("rfrly: data-rfid required");

  // Verify Snitcher only if SNID is provided
  if (SNID) {
    const isNumericSnid = /^\d+$/.test(SNID);
    if (isNumericSnid) {
      // Legacy Snitcher verification
      window.snid && window.snid("verify", SNID);
    } else {
      // New Radar/Snitcher - initialization happens automatically
      // The Snitcher namespace is already set up with profileId
    }
  }

  /* ---- 2. Helpers ---- */
  const uuid = () =>
    self.crypto && typeof self.crypto.randomUUID === "function"
      ? self.crypto.randomUUID()
      : Math.random().toString(36).slice(2) + Date.now();

  const LS_DEVICE = "rfly_device";
  const SS_SESSION = "rfly_session";
  const device_uuid = (localStorage[LS_DEVICE] ||= uuid());
  const session_uuid = (sessionStorage[SS_SESSION] ||= uuid());

  /* timers */
  const t0 = Date.now();
  let engaged = 0,
    mark = t0;
  const resume = () => (mark = Date.now());
  const pause = () => (engaged += Date.now() - mark);
  addEventListener("visibilitychange", () =>
    document.visibilityState === "hidden" ? pause() : resume()
  );
  addEventListener("blur", pause);
  addEventListener("focus", resume);

  const dissect = (u) => {
    const x = new URL(u);
    return {
      url: x.href,
      scheme: x.protocol.slice(0, -1),
      host: x.hostname,
      path: x.pathname,
      query: Object.fromEntries(x.searchParams),
      fragment: x.hash.slice(1),
    };
  };

  const page = dissect(location.href);
  const ref = document.referrer ? dissect(document.referrer) : null;
  const utm = ["source", "medium", "campaign", "content", "term"].reduce(
    (o, k) =>
      page.query[`utm_${k}`] ? { ...o, [k]: page.query[`utm_${k}`] } : o,
    {}
  );

  const base = () => ({
    snid: SNID,
    rfid: RFID,
    event_type: "page_visit",
    event_uuid: uuid(),
    created_at: new Date().toISOString(),
    context: {
      type: "web_session",
      geo: {},
      session: {
        uuid: session_uuid,
        start_time: new Date(t0).toISOString(),
        last_activity: new Date().toISOString(),
        total_time: 0,
        engaged_time: 0,
      },
      device_uuid,
      user_agent: navigator.userAgent,
    },
    user_properties: { $uuid: device_uuid },
    event_properties: {},
    meta: { library: { name: "radar.js", version: "1.0.42" } },
  });

  const send = (body) => {
    const blob = new Blob([JSON.stringify(body)], { type: "application/json" });
    (navigator.sendBeacon && navigator.sendBeacon(COLLECT, blob)) ||
      fetch(COLLECT, { method: "POST", body: blob, keepalive: true });
  };

  /* first ping */
  send(base());

  /* final ping */
  addEventListener("pagehide", () => {
    pause();
    const ev = base();
    ev.event_type = "page_leave";
    ev.context.session.total_time = Date.now() - t0;
    ev.context.session.engaged_time = engaged;
    ev.context.session.last_activity = new Date().toISOString();
    if (ref) ev.context.referrer = ref;
    if (Object.keys(utm).length) ev.context.utm = utm;
    ev.event_properties = {
      ...page,
      $total_time_on_page: ev.context.session.total_time,
      $engaged_time_on_page: engaged,
    };
    send(ev);
  });
})();
