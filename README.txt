My submission is programmed in python 3.8.10 on the drexel tux linux servers.

My source code is in CG_hw2.py, but I put my code in an executable file call CG_hw2 that can be called
as needed in the requirements.

I didn't change much from the original code other than having to handle the new commands lineto and moveto.

Coordinates:
This class is within lines 8 - 21. I just pulled out the method I had previously in two separate classes
and made it compatible with the new commands.

Clip(lines, args):
I added a few more helper function and the main clipping algorithm to this class.

    Lines 140 - 178
    sutherland_hodgman_clipping() - Performs the clipping of the input polygon. I first define the clipping edges. Then I
                                    assign the first polygon. I initialize two variables all_outside and vertices that are
                                    responsible for checking if all vertices will be outside of the world-view. This way
                                    I get no errors when trying to grab edges on the next iteration.

                                    I then begin to follow the algorithm stated in class. I loop through each clipping edge, find 
                                    out of if the vertices for the current edge of the polygon are "inside" or "outside", perform
                                    the necessary intersection calculations and append the correct vertices to the clipped_polygon 
                                    list depending on the case. I then update the polygon and loop through again untill there is no
                                    more clipping edges.

    Lines 249 - 261
    _is_inside(vertex, edge) -  Takes in a vertex and sees if that point is "inside" the clipping edges bounds.

    Lines 263 - 275
    _compute_intersection(v1, v2, edge) -   Takes in the current working edge of the polygon and finds the point of intersection
                                            for the current clipping edge.

    Lines 277 288
    _set_edges(x_min, y_min, x_max, y_max) -    Takes in word view arguments and creates the lines responsible for each edge.

