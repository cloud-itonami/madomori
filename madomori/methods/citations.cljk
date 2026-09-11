;; madomori 窓守 — provenance for the safety-gate constants (data/citations.edn).
;;
;; The two ★ gates REFUSE work by comparing against numbers. This ns answers
;; "what grounds that number?" so the refusal can be audited instead of trusted.
;;
;; The load-bearing function is `cited?`, and it is deliberately PESSIMISTIC:
;; a subject with no record is UNCITED, never "presumed fine". A provenance
;; checker that answers "no problem" for things it has never seen would report
;; unmeasured as clean — the failure this repo's gates exist to avoid.
;;
;; Pure Clojure, no deps → babashka-runnable AND kotoba-pywasm-portable.
;; Per ADR-2606142020 (madomori R0).
(ns madomori.methods.citations
  (:require [clojure.edn :as edn]))

(def ^:const default-path "data/citations.edn")

(defn load-citations
  "Read the provenance record. RAISES if it is empty — an empty provenance file
   and a missing one must not answer the same as a satisfied one (a checker that
   returns 'nothing uncited' because it read nothing is the bug it guards)."
  ([] (load-citations default-path))
  ([path]
   (let [content (edn/read-string (slurp path))]
     (when-not (and (vector? content) (seq content))
       (throw (ex-info "citation record is empty or malformed — refusing to report provenance"
                       {:path path})))
     content)))

(defn by-id
  "The citation entry with `id`, or nil."
  [citations id]
  (first (filter #(= id (:citation/id %)) citations)))

(defn regulatory
  "Entries that quote a fetched instrument (they carry :citation/url + :citation/quote)."
  [citations]
  (filterv #(and (:citation/url %) (:citation/quote %)) citations))

(defn gaps
  "Entries recording that a constant has NO verified source. These are findings,
   not omissions: they were searched for and not found."
  [citations]
  (filterv #(#{:uncited-assumption :design-decision :representative-data} (:citation/kind %))
           citations))

(defn cited?
  "True iff `id` names an entry that quotes a fetched instrument. A gap entry is
   NOT cited (that is the whole point of recording it), and an unknown id is NOT
   cited — absence of a record is absence of provenance."
  [citations id]
  (boolean (when-let [c (by-id citations id)]
             (and (:citation/url c) (:citation/quote c) (not (:citation/kind c))))))

(defn grounds
  "What `id` establishes, as a vector of statements. RAISES on an unknown id —
   asking about a constant nobody recorded must surface, not return empty."
  [citations id]
  (let [c (by-id citations id)]
    (when (nil? c)
      (throw (ex-info "unknown citation id — no provenance recorded" {:id id})))
    (vec (:citation/grounds c))))

(defn does-not-ground
  "What `id` explicitly does NOT establish. Reading this is not optional: several
   instruments here mandate a work-stop while stating no threshold, so using them
   to justify a number would be fabricated provenance."
  [citations id]
  (let [c (by-id citations id)]
    (when (nil? c)
      (throw (ex-info "unknown citation id — no provenance recorded" {:id id})))
    (vec (:citation/does-not-ground c))))

(defn report
  "Non-raising provenance summary for the R0 report."
  [citations]
  {:citation-count (count citations)
   :regulatory-count (count (regulatory citations))
   :gap-count (count (gaps citations))
   :cited (mapv :citation/id (regulatory citations))
   :uncited (mapv :citation/id (gaps citations))})

(defn -main
  "Print the provenance record: what is grounded, and what is admitted uncited."
  [& _]
  (let [cs (load-citations)
        r (report cs)]
    (println "-- madomori provenance (data/citations.edn) --")
    (println)
    (doseq [c (regulatory cs)]
      (println "●" (name (:citation/id c)) "—" (:citation/instrument c))
      (println "   " (:citation/url c) " [HTTP" (:citation/http-status c) "verified" (:citation/verified-at c) "]")
      (doseq [g (:citation/grounds c)] (println "    grounds:" g))
      (doseq [g (:citation/does-not-ground c)] (println "    does NOT ground:" g))
      (println))
    (println "-- admitted uncited (searched, not found) --")
    (doseq [c (gaps cs)]
      (println "○" (name (:citation/id c)) "—" (:citation/subject c))
      (println "   " (:citation/status c))
      (println))
    (println (format "%d entries: %d quote a fetched instrument, %d record a gap."
                     (:citation-count r) (:regulatory-count r) (:gap-count r)))))
