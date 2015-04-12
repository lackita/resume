<pre><code class="java">package name.colinwilliams.dotcollision;

import java.awt.Point;
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;

/* Being able to quickly look up dots proximity is a challenge.
 * An ArrayList would require O(n), but a hash lookup is difficult
 * because it works with discrete values.  To make this possible, we'll
 * divide the field into squares the same size as the dots, tracking all
 * of the dots that have any part in the square.  This limits lookup by
 * only having to consider dots that share a square. */
public class DotCollider implements Iterable<Dot> {
    private HashSet<Dot> dots;
    private HashMap<Point, HashSet<Dot>> grid;
    public DotCollider() {
	dots = new HashSet<Dot>();
	grid = new HashMap<Point, HashSet<Dot>>();
    }

    @Override
    public Iterator<Dot> iterator() {
	return dots.iterator();
    }

    public void add(Dot dot) {
	dots.add(dot);
	for (HashSet<Dot> covered_square : covered_squares(dot))
	    covered_square.add(dot);
    }

    public HashSet<Dot> dots_within(Dot dot) {
	HashSet<Dot> all_dots = new HashSet<Dot>();
	for (HashSet<Dot> dots_within_square : covered_squares(dot))
	    for (Dot overlap_candidate : dots_within_square)
		if (dot.overlaps(overlap_candidate))
		    all_dots.add(overlap_candidate);
	return all_dots;
    }

    public void update_location(Dot dot) {
	for (Point grid_box_corner : dot.previous_dot().square_corners_covering()) {
	    if (grid.containsKey(grid_box_corner)) grid.get(grid_box_corner).remove(dot);
	}
	add(dot);
    }

    private ArrayList<HashSet<Dot>> covered_squares(Dot dot) {
	ArrayList<HashSet<Dot>> result = new ArrayList<HashSet<Dot>>();
	for (Point corner : dot.square_corners_covering()) {
	    ensure_corner_exists(result, corner);
	    result.add(grid.get(corner));
	}
	return result;
    }

    private void ensure_corner_exists(ArrayList<HashSet<Dot>> result, Point corner) {
	if (!grid.containsKey(corner)) grid.put(corner, new HashSet<Dot>());
    }

    public Collection<Dot> get_collisions(Dot d) {
	ArrayList<Dot> collisions = new ArrayList<Dot>();
	for (Dot collided : dots_within(d)) {
	    if (d != collided)
		collisions.add(collided);
	}
	return collisions;
    }
}</code></pre>
