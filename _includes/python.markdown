<pre><code class="python">import unittest

class UlamTest(unittest.TestCase):
    def test_precomputed(self):
        u = U(5)
        for (k, x) in [(1, 2), (2, 5), (3, 7), (4, 9), (5, 11), (6, 12), (7, 13), (8, 15), (9, 19), (10, 23), (11, 27), (12, 29)]:
            self.assertEquals(u.compute(k), x)

    def test_period(self):
        for n in range(2, 5):
            v = 2*n + 1
            u = U(v)
            p = Period(u, 100)
            ks = [500*x + 500 for x in range(1, 20)]
            ks.append(p.period_start + 1)
            for k in ks:
                self.assertEquals(u.compute(k), p.compute(k), "v: " + str(v) + " k: " + str(k))

    def test_large_values(self):
        self.assertEquals(Period(U(5), 100).compute(10**11), 393749999981)
        self.assertEquals(Period(U(7), 100).compute(10**11), 484615384605)
        self.assertEquals(Period(U(9), 100).compute(10**11), 400450450395)
        self.assertEquals(Period(U(11), 100).compute(10**11), 399877149781)
        self.assertEquals(Period(U(13), 100).compute(10**11), 399966136001)
        self.assertEquals(Period(U(15), 100).compute(10**11), 637499999951)
        self.assertEquals(Period(U(17), 100).compute(10**11), 400001574629)
        self.assertEquals(Period(U(19), 100).compute(10**11), 399999473477)
        self.assertEquals(Period(U(21), 100).compute(10**11), 399999900065)

if __name__ == '__main__':
    unittest.main()


from Queue import PriorityQueue

class U(object):
    def __init__(self, v):
        self.computed = []
        self.candidates = PriorityQueue()
        self.candidates.put(2)
        self.candidates.put(v)
        self.candidates.put(2*v + 2)

    def compute(self, k):
        while len(self.computed) < k:
            candidate = self.candidates.get()
            valid = True
            while not self.candidates.empty() and self.candidates.queue[0] == candidate:
                valid = False
                self.candidates.get()
            if valid:
                self.computed.append(candidate)
                if candidate % 2 == 1:
                    self.candidates.put(2 + candidate)
                    self.candidates.put(2*self.computed[1] + 2 + candidate)
        return self.computed[k - 1]


class UTest(unittest.TestCase):
    def test_precomputed(self):
        u = U(5)
        for (k, x) in [(1, 2), (2, 5), (3, 7), (4, 9), (5, 11), (6, 12), (7, 13), (8, 15), (9, 19), (10, 23), (11, 27), (12, 29)]:
            self.assertEquals(u.compute(k), x)

if __name__ == '__main__':
    unittest.main()


class Period(object):
    def __init__(self, u, period_start):
        self.period_start = period_start
        self.minimum_period = 20
        self.diffs = []
        self.previous = u.compute(period_start)
        self.u = u
        self.period = self.compute_period()

    def compute_period(self):
        period_length = self.minimum_period
        while not self.verify_period(period_length):
            period_length += 1
        return self.diffs[0:period_length]

    def verify_period(self, period_length):
        i = period_length
        while i <= 2*period_length:
            if self.lookup_diff(i) != self.lookup_diff(i % period_length):
                return False
            i += 1
        return True

    def lookup_diff(self, i):
        while i >= len(self.diffs):
            s = self.u.compute(self.period_start + 1 + len(self.diffs))
            self.diffs.append(s - self.previous)
            self.previous = s
        return self.diffs[i]

    def compute(self, k):
        if k <= self.period_start:
            return self.u.compute(k)
        fundamental_difference = sum(self.period)
        base = self.u.compute(self.period_start) + fundamental_difference*((k - self.period_start)/len(self.period))
        return base + sum(self.period[0:(k - self.period_start)%len(self.period)])

result = 0
for n in range(2, 11):
    v = 2*n + 1
    u = U(v)
    period = Period(u, 100)
    x = period.compute(10**11)
    print str(v) + ": " + str(x)
    print len(period.period)
    result += x
print result</code></pre>
