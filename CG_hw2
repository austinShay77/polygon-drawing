#!/usr/bin/env python3

import argparse
import math
import sys


class Coordinates:
    def __init__(self):
        self.x1 = 0
        self.y1 = 0
        self.x2 = 0
        self.y2 = 0

    # uses the current line to re-assign attributes
    def _set_points(self, line):
        if "Line" in line:
            self.x1, self.y1 = float(line[0]), float(line[1])
            self.x2, self.y2 = float(line[2]), float(line[3])
        elif "moveto" in line or "lineto" in line:
            self.x1, self.y1 = float(line[0]), float(line[1])

class FileIO:
    def __init__(self, path):
        self.path = path

    # take a list of lines, write them to stdout in .ps form
    def write(self, lines, args):
        x_size = args.upper_boundx - args.lower_boundx + 1
        y_size = args.upper_boundy - args.lower_boundy + 1

        is_moveto = True

        sys.stdout.write(f"%%BeginSetup\n   << /PageSize [{x_size} {y_size}] >> setpagedevice\n%%EndSetup\n\n%%%BEGIN")
        for line in lines:
            if "Line" in line:
                sys.stdout.write(f"\n{line[0]-args.lower_boundx} {line[1]-args.lower_boundy} moveto\n{line[2]-args.lower_boundx} {line[3]-args.lower_boundy} lineto\nstroke")
            elif "moveto" in line or "lineto" in line:
                sys.stdout.write(f"\n{line[0]-args.lower_boundx} {line[1]-args.lower_boundy} {line[2]}")
            elif "stroke" in line:
                sys.stdout.write(f"\n{line[0]}")
            else:
                if is_moveto:
                    sys.stdout.write(f"\n{line[0]-args.lower_boundx} {line[1]-args.lower_boundy} moveto")
                    is_moveto = False
                else:
                    sys.stdout.write(f"\n{line[0]-args.lower_boundx} {line[1]-args.lower_boundy} lineto")
        sys.stdout.write(f"\n%%%END\n")

    # take a .ps file, parse it into a 2d array
    def read(self):
        with open(self.path) as file:
            commands = self._find_meaningful_lines([line.rstrip() for line in file if line != "\n"])
        return self._split_lines(commands)
    
    # splites the line so each coordinate is its own element
    def _split_lines(self, commands):
        organized = [element.split() for element in commands]
        return organized

    # only keeps lines after %%%BEGIN and before %%%END
    def _find_meaningful_lines(self, commands):
        meaningless = True
        meaningful_lines = []
        for element in commands:
            if meaningless:
                if f"%%%BEGIN" in element:
                    meaningless = False
            else:
                if f"%%%END" in element:
                    break
                meaningful_lines.append(element)
        return meaningful_lines

class Transformer(Coordinates):
    def __init__(self, lines, args):
        super().__init__()
        self.lines = lines
        self.args = args

    # create a new list of transformed lines
    def transform_lines(self):
        new_lines = []
        for line in self.lines:
            if "stroke" in line:
                new_lines.append(line)
            else:
                self._set_points(line)
                self._scale()
                self._rotate()
                self._translate()
                if "Line" in line:
                    new_lines.append([self.x1, self.y1, self.x2, self.y2, line[4]])
                elif "moveto" in line or "lineto" in line:
                    new_lines.append([self.x1, self.y1, line[2]])
        return new_lines

    def _scale(self):
        self.x1 = self.x1 * self.args.scaling_factor
        self.y1 =  self.y1 * self.args.scaling_factor

        self.x2 = self.x2 * self.args.scaling_factor
        self.y2 =  self.y2 * self.args.scaling_factor

    def _rotate(self):
        phi = self.args.ccr * math.pi / 180
        x1 = self.x1 * math.cos(phi) - self.y1 * math.sin(phi)
        y1 = self.x1 * math.sin(phi) + self.y1 * math.cos(phi)
        x2 = self.x2 * math.cos(phi) - self.y2 * math.sin(phi)
        y2 = self.x2 * math.sin(phi) + self.y2 * math.cos(phi)

        self.x1 = x1
        self.y1 = y1
        self.x2 = x2
        self.y2 = y2
    
    def _translate(self):
        self.x1 += self.args.x_dim
        self.y1 += self.args.y_dim

        self.x2 += self.args.x_dim
        self.y2 += self.args.y_dim
            

class Clip(Coordinates):
    def __init__(self, lines, args):
        super().__init__()
        self.lines = lines
        self.args = args
        self.inside = 0 # 0000
        self.left = 1   # 0001
        self.right = 2  # 0010
        self.bottom = 4 # 0100
        self.top = 8    # 1000
        self.x_min = self.args.lower_boundx
        self.y_min = self.args.lower_boundy
        self.x_max = self.args.upper_boundx
        self.y_max = self.args.upper_boundy

    def sutherland_hodgman_clipping(self):                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
        edges = self._set_edges(self.x_min, self.y_min, self.x_max, self.y_max)
        p = self.lines
        all_outside = 1
        vertices = len(p) - 1
        # [[left], [bottom], [right], [top]]
        for edge in edges:
            clipped_polygon = []
            for index, line in enumerate(p):
                if all_outside == vertices:
                    return []
                if index+1 < len(p):
                    if "stroke" in p[index+1]:
                        clipped_polygon.append(clipped_polygon[0])
                    else:
                        v1 = [line[0], line[1]]
                        v2 = [p[index+1][0], p[index+1][1]]

                        is_v1_inside = self._is_inside(v1, edge)
                        is_v2_inside = self._is_inside(v2, edge)

                        # both in
                        if is_v1_inside and is_v2_inside:
                            clipped_polygon.append(v2)
                        # both out
                        elif not is_v1_inside and not is_v2_inside:
                            all_outside += 1
                        # v1 in v2 out
                        elif is_v1_inside is True and is_v2_inside is False:
                            inside_outside = self._compute_intersection(v1, v2, edge)
                            clipped_polygon.append(inside_outside)
                        # v1 out v2 in
                        elif is_v1_inside is False and is_v2_inside is True:
                            outside_inside = self._compute_intersection(v1, v2, edge)
                            clipped_polygon.append(outside_inside)
                            clipped_polygon.append(v2)
            clipped_polygon.append(["stroke"])
            p = clipped_polygon
        return clipped_polygon

    # sees which lines needs to be clipped and re-calculates coordinates
    def cohen_sutherland_clipping(self):
        clipped_lines = []
        for line in self.lines:
            self._set_points(line)
            x = 0
            y = 0

            p1_code = self._find_code(self.x1, self.y1)
            p2_code = self._find_code(self.x2, self.y2)

            # both in
            if p1_code == 0 and p2_code == 0:
                clipped_lines.append([self.x1, self.y1, self.x2, self.y2, "Line"])
            # both out
            elif (p1_code & p2_code) != 0:
                continue
            else:
                if p1_code == 0:
                    out_code = p2_code
                else:
                    out_code = p1_code

                if out_code == self.left:
                    x = self.x_min
                    y = (self.x_min - self.x1)/(self.x2 - self.x1) * (self.y2 - self.y1) + self.y1
                elif out_code == self.right:
                    x = self.x_max
                    y = (self.x_max - self.x1)/(self.x2 - self.x1) * (self.y2 - self.y1) + self.y1
                elif out_code == self.bottom:
                    y = self.y_min
                    x = (self.y_min - self.y1)/(self.y2 - self.y1) * (self.x2 - self.x1) + self.x1
                elif out_code == self.top:
                    y = self.y_max
                    x = (self.y_max - self.y1)/(self.y2 - self.y1) * (self.x2 - self.x1) + self.x1

                if out_code == p1_code:
                    self.x1 = x
                    self.y1 = y 
                else:
                    self.x2 = x
                    self.y2 = y 
                clipped_lines.append([self.x1, self.y1, self.x2, self.y2, "Line"])
        return clipped_lines

    # calculates the binary value of clipped lines
    def _find_code(self, x, y):
        code = self.inside
        if x < self.x_min:
            code |= self.left
        elif x > self.x_max:
            code |= self.right
        if y < self.y_min:
            code |= self.bottom
        elif y > self.y_max:
            code |= self.top
        return code

    # vertex points: 
    #       x = vertex[0]
    #       y = vertex[1]
    # edge points:
    #   A:
    #       x = edge[0]
    #       y = edge[1]  
    #   B:
    #       x = edge[2]
    #       y = edge[3]
    # vertices are given in counter-clockwise order so "inside" is on the left of the edge
    def _is_inside(self, vertex, edge):
        c = (vertex[0] - edge[0]) * (edge[3] - edge[1]) - (vertex[1] - edge[1]) * (edge[2] - edge[0])
        if "right" in edge or "bottom" in edge:
            c *= -1
        # in/left of line
        if c > 0:
            return True
        # out/right of line
        if c < 0:
            return False
        # on edge
        if c == 0:
            return False

    def _compute_intersection(self, v1, v2, edge):
        new_vertex = []
        x1, y1 = v1[0], v1[1]
        x2, y2 = v2[0], v2[1]
        x3, y3 = edge[0], edge[1]
        x4, y4 = edge[2], edge[3]

        x = ( (x1*y2 - y1*x2) * (x3 - x4) - (x1 - x2) * (x3*y4 - y3*x4) ) / ( (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4) )
        y = ( (x1*y2 - y1*x2) * (y3 - y4) - (y1 - y2) * (x3*y4 - y3*x4) ) / ( (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4) )
        new_vertex.append(x)
        new_vertex.append(y)

        return new_vertex

    def _set_edges(self, x_min, y_min, x_max, y_max):
        edges = []
        # right
        edges.append([x_max, y_min, x_max, y_max, "right"])
        # bottom
        edges.append([x_min, y_min, x_max, y_min, "bottom"])
        # left
        edges.append([x_min, y_min, x_min, y_max, "left"])
        # top
        edges.append([x_min, y_max, x_max, y_max, "top"])

        return edges

def hw2(args):
    fileio = FileIO(args.ps_file)
    lines = fileio.read()

    # print(lines)

    transformer = Transformer(lines, args)
    new_lines = transformer.transform_lines()

    # print(new_lines)

    clipping = Clip(new_lines, args)
    # clipped_lines = clipping.cohen_sutherland_clipping()
    # print(clipped_lines)
    clipped_polygons = clipping.sutherland_hodgman_clipping()
    # print(clipped_polygons)

    fileio.write(clipped_polygons, args)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("-f", "--ps_file", type=str, default="hw2_a.ps")
    parser.add_argument("-s", "--scaling_factor", type=float, default=1.0)
    parser.add_argument("-r", "--ccr", type=int, default=0)
    parser.add_argument("-m", "--x_dim", type=int, default=0)
    parser.add_argument("-n", "--y_dim", type=int, default=0)

    parser.add_argument("-a", "--lower_boundx", type=int, default=0)
    parser.add_argument("-b", "--lower_boundy", type=int, default=0)
    parser.add_argument("-c", "--upper_boundx", type=int, default=499)
    parser.add_argument("-d", "--upper_boundy", type=int, default=499)

    args = parser.parse_args()

    hw2(args)

if __name__ == "__main__":
    main()