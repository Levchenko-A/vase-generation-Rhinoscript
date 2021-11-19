import rhinoscriptsyntax as rs
import random as rnd

def draw_vase(basis_curve, Height):
    """function for creating a vase, based on a circle from input
        and precoded relationships"""
        
    # create a second section dor durther loft
    translation = (0,0,0.33*Height)
    second_section=rs.CopyObject(basis_curve,translation)
    second_section=rs.ScaleObject(second_section,rs.CircleCenterPoint(second_section),(1.25,1.25,1.25))
    
    # create a third section dor durther loft
    translation = (0,0,0.66*Height)
    third_section=rs.CopyObject(basis_curve,translation)
    third_section=rs.ScaleObject(third_section,rs.CircleCenterPoint(third_section),(.5,.5,.5))
    
    # create a fourth section dor durther loft
    translation = (0,0,Height)
    fourth_section=rs.CopyObject(basis_curve,translation)
    fourth_section=rs.ScaleObject(fourth_section,rs.CircleCenterPoint(fourth_section),(.75,.75,.75))
    points_list = [basis_curve,second_section,third_section,fourth_section]
    points_list.reverse()
    
    # create a surface
    vase_body = rs.AddLoftSrf(points_list)
    return vase_body
    
def u_v_points_dict(surface,intervals):
    """fuction for creating two dictionaries - points that are 
        achieved from bone-surface and points that are normals to first points"""
    
    u_v_dict={}
    norm_points_dict = {}
    
    u_domain = rs.SurfaceDomain(surface,0)
    v_domain = rs.SurfaceDomain(surface,1)
    
    step_u = (u_domain[1]-u_domain[0])/intervals
    step_v = (v_domain[1]-v_domain[0])/intervals
    
    for i in range(intervals+1):
        for j in range(intervals+1):
            
            u = u_domain[0]+step_u*i
            v = v_domain[0]+step_v*j
            
            #getting points from surface
            point = rs.EvaluateSurface(surface,u,v)
            u_v_dict[(i,j)]= point
            
            #getting normal points
            norm_vec = rs.SurfaceNormal(surface,(u,v))
            norm_vec = rs.VectorScale(norm_vec, .5)
            norm_vec = rs.PointAdd(norm_vec,point)
            norm_points_dict[(i,j)] = norm_vec
            
    return u_v_dict,norm_points_dict

def render_color(object,I,J,intervals):
    """fuction for creating a color gradient for rendering"""
    
    mat_index = rs.AddMaterialToObject(object)
    rs.MaterialColor(mat_index,(255/intervals*I,255-(255/intervals)*J,255))

def centroid_point(x,y,points_dictionary,surface):
    """function for getting a centroid for building 3D structures in pattern"""
    
    new_curve = rs.AddCurve((points_dictionary[(x-1,y-1)],points_dictionary[(x,y)]))
    mid_point=rs.CurveMidPoint(new_curve)
    centr_point = rs.BrepClosestPoint(surface,mid_point)[0]
    rs.DeleteObject(new_curve)
    return centr_point

def creating_pattern(points_dict,norm_points_dict,intervals,surface):
    """function for creating a pattern"""
    
    centroids_dict={}
    for i in range(intervals+1):
        for j in range(intervals+1):
            if i>0 and j>0:
                
                centroid = centroid_point(i,j,points_dict,surface)
                centroids_dict[(i,j)]=(centroid)
                
                # creating curves that should lay on bone-surface
                curve_1=rs.AddCurve((points_dict[(i-1,j-1)],centroid,points_dict[(i-1,j)]),3)
                curve_2=rs.AddCurve((points_dict[(i-1,j)],centroid,points_dict[(i,j)]),3)
                curve_3=rs.AddCurve((points_dict[(i,j)],centroid,points_dict[(i,j-1)]),3)
                curve_4=rs.AddCurve((points_dict[(i,j-1)],centroid,points_dict[(i-1,j-1)]),3)
                surf_curve = rs.JoinCurves((curve_1,curve_2,curve_3,curve_4),delete_input = True)
                rs.ReverseCurve(surf_curve)
                
                # creating curves that that are based on normals to surface
                curve_1=rs.AddCurve((norm_points_dict[(i-1,j-1)],centroid,norm_points_dict[(i-1,j)]),3)
                curve_2=rs.AddCurve((norm_points_dict[(i-1,j)],centroid,norm_points_dict[(i,j)]),3)
                curve_3=rs.AddCurve((norm_points_dict[(i,j)],centroid,norm_points_dict[(i,j-1)]),3)
                curve_4=rs.AddCurve((norm_points_dict[(i,j-1)],centroid,norm_points_dict[(i-1,j-1)]),3)
                
                norm_curve = rs.JoinCurves((curve_1,curve_2,curve_3,curve_4),delete_input = True)
                rs.ReverseCurve(norm_curve)
                
                #creating basis surfaces for our pattern
                polysurf=rs.AddLoftSrf((norm_curve,surf_curve))
                render_color(polysurf,i,j,intervals)
                
    # creating horizontal pattern elements (patches between basis surfaces) 
    for i in range(intervals):
        for j in range(intervals+1):
            if i>0 and j>0:
                curve_1 = rs.AddCurve((points_dict[(i,j-1)],centroids_dict[(i,j)],points_dict[(i,j)]),3)
                curve_2 = rs.AddCurve((points_dict[(i,j-1)],centroids_dict[(i+1,j)],points_dict[(i,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_1,curve_2))
                render_color(patch_surf,i,j,intervals)
                
                curve_1 = rs.AddCurve((norm_points_dict[(i,j-1)],centroids_dict[(i,j)],norm_points_dict[(i,j)]),3)
                curve_2 = rs.AddCurve((norm_points_dict[(i,j-1)],centroids_dict[(i+1,j)],norm_points_dict[(i,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_1,curve_2))
                render_color(patch_surf,i,j,intervals)
                
    # creating vertical pattern elements (patches between basis surfaces)
    for i in range(intervals+1):
        for j in range(intervals):
            if i>0 and j>0:
                curve_1 = rs.AddCurve((points_dict[(i,j)],centroids_dict[(i,j)],points_dict[(i-1,j)]),3)
                curve_2 = rs.AddCurve((points_dict[(i,j)],centroids_dict[(i,j+1)],points_dict[(i-1,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_1,curve_2))
                render_color(patch_surf,i,j,intervals)
                
                curve_1 = rs.AddCurve((norm_points_dict[(i,j)],centroids_dict[(i,j)],norm_points_dict[(i-1,j)]),3)
                curve_2 = rs.AddCurve((norm_points_dict[(i,j)],centroids_dict[(i,j+1)],norm_points_dict[(i-1,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_1,curve_2))
                render_color(patch_surf,i,j,intervals)
                
            if i>0 and j==0:
                curve_1 = rs.AddCurve((points_dict[(i,j)],centroids_dict[(i,j+1)],points_dict[(i-1,j)]),3)
                curve_2 = rs.AddCurve((points_dict[(i,j)],centroids_dict[(i,intervals)],points_dict[(i-1,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_2,curve_1))
                render_color(patch_surf,i,j,intervals)
                
                curve_1 = rs.AddCurve((norm_points_dict[(i,j)],centroids_dict[(i,j+1)],norm_points_dict[(i-1,j)]),3)
                curve_2 = rs.AddCurve((norm_points_dict[(i,j)],centroids_dict[(i,intervals)],norm_points_dict[(i-1,j)]),3)
                patch_surf = rs.AddEdgeSrf((curve_2,curve_1))
                render_color(patch_surf,i,j,intervals)
                
def main():
    rs.EnableRedraw(False)
    
    #get input information
    basis_circle = rs.GetObject("Select the basic circle",rs.filter.curve)
    height_rel = rs.GetReal("input the vase height relation to the base radius")
    height = height_rel*rs.CircleRadius(basis_circle)
    Interval = rs.GetInteger("input the number of intervals")
    
    # create surface that will be used as a bone structure
    vase = draw_vase(basis_circle,height)
    u_v_points=u_v_points_dict(vase, Interval)[0]
    u_v_norm_points = u_v_points_dict(vase, Interval)[1]
    
    # create a pattern and delete surface (bone structure)
    creating_pattern(u_v_points,u_v_norm_points, Interval,vase)
    rs.DeleteObject(vase)
        
    # delete auxiliary geometry (curves and points)
    all_objects = rs.AllObjects()
    for object in all_objects:
        if rs.IsPoint(object) or rs.IsCurve(object):
            rs.DeleteObject(object)
    rs.EnableRedraw(True)
            
main()
