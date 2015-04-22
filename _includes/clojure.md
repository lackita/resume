Problem
=======
[Problem #140](http://www.4clojure.com/problem/140) on (4clojure.org) asks to build a Karnaugh Map to simplify provided boolean 
expressions.  They will be provided in the following form:

<pre><code class="clojure">#{#{'a 'B 'C 'd}
  #{'A 'b 'c 'd}
  #{'A 'b 'c 'D}
  #{'A 'b 'C 'd}
  #{'A 'b 'C 'D}
  #{'A 'B 'c 'd}
  #{'A 'B 'c 'D}
  #{'A 'B 'C 'd}}</code></pre>

Where each set is a rule, uppercase letters represent true and lowercase false.

Solution
========
There are three basic components to the problem:
- Build a data structure representing the Karnaugh Map for the rules.
- Determinine the boxes to draw around groups of true entries.
- Remove superfluous boxes.

<pre><code class="clojure">(defn simplify-rules [conditions]
  (->> (build-kmap conditions)
       identify-simplifications
       (prune-duplicates conditions)))</code></pre>

Building the K-Map
------------------


<pre><code class="clojure">(defn negative [s]
  (symbol (.toLowerCase (name s))))

(defn positive [s]
  (symbol (.toUpperCase (name s))))

(defn gray-codes [symbols]
  (if (empty? symbols) [#{}]
      (let [varying (first symbols)
            subsequent-gray-codes (gray-codes (rest symbols))]
        (concat (map #(conj % (negative varying))
                     subsequent-gray-codes)
                (map #(conj % (positive varying))
                     (reverse subsequent-gray-codes))))))

(defn divide-symbols [symbols]
  (split-at (Math/ceil (/ (count symbols) 2)) symbols))

(defn build-grid [conditions x-gray-codes y-gray-codes]
  (map (fn [y-gray-code]
         (map #(if (conditions (set (concat % y-gray-code))) 1 0)
              x-gray-codes)) y-gray-codes))

(defn build-kmap [conditions]
  (let [[x-gray-codes y-gray-codes] (->> (first conditions)
                                         (map negative)
                                         sort
                                         divide-symbols
                                         (map gray-codes))
        grid (build-grid conditions x-gray-codes y-gray-codes)]
    {:grid grid
     :x-gray-codes x-gray-codes
     :y-gray-codes y-gray-codes
     :width (count (first grid))
     :height (count grid)}))</code></pre>

Determine Boxes
---------------


<pre><code class="clojure">(defn gray-codes-for-box [gray-codes positions]
  (apply clojure.set/intersection
         (map #(nth gray-codes %) positions)))

(defn cyclic-ranges [distance size]
  (for [start (range distance)]
    (take size (drop start (cycle (range distance))))))

(defn rectangle-coordinates [kmap size]
  (set (for [divisor [1 2 4]
             xs (cyclic-ranges (kmap :width)
                               (/ size divisor))
             ys (cyclic-ranges (kmap :height)
                               divisor)]
         (set (for [x xs y ys] [x y])))))

(defn convert-pairs-to-expressions [kmap pairs]
  [(clojure.set/union
    (gray-codes-for-box (kmap :x-gray-codes)
                        (map first pairs))
    (gray-codes-for-box (kmap :y-gray-codes)
                        (map second pairs)))
   (set (map (fn [[x y]] (nth (nth (kmap :grid)
                                   y) x))
             pairs))])

(defn boxes-sized [kmap size]
  (set (map #(convert-pairs-to-expressions kmap %)
            (rectangle-coordinates kmap size))))

(defn identify-simplifications [kmap]
  (map (fn [size] (->> (boxes-sized kmap size)
                       (filter #(not (get-in % [1 0])))
                       (map first)))
       [8 4 2 1]))</code></pre>

Prune Duplicates
----------------
<pre><code class="clojure">(defn already-fulfilled? [existing-conditions new-condition]
  (not (some (fn [existing]
               (every? new-condition existing))
             existing-conditions)))

(defn prune-fulfilled [simplified-conditions]
  (set (reduce (fn [existing-conditions new-conditions]
                 (into existing-conditions
                       (filter #(already-fulfilled? existing-conditions %) new-conditions)))
               simplified-conditions)))

(defn identify-condition-coverage [full-conditions simplified-conditions]
  (into {} (map (fn [condition]
                  [condition (filter #(every? condition %)
                                     simplified-conditions)])
                full-conditions)))

(defn prune-duplicates [full-conditions simplified-conditions]
  (->> (prune-fulfilled simplified-conditions)
       (identify-condition-coverage full-conditions)
       (filter #(= (count (val %)) 1))
       (mapcat val)
       set))</code></pre>
