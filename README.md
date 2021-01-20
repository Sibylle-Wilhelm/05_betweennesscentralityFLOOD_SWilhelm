# 05_betweennesscentralityFLOOD_SWilhelm
Exercise Betweenness Centrality flooded situation

#script for analysing betweenness centrality in the road network of the Canton of ZH
import numpy as np
import pandas as pd
import networkx as nx
import geopandas as gpd
import matplotlib.pyplot as plt


#general workspace settings
myworkspace="C:/Users/Sibylle Wilhelm/Desktop/Master/HS20/Introduction to network science/Exercises Python/Ex_ZH" 

#input data: the csv file for nodes and edges
nodesfile = myworkspace+"/zh_nodes.shp"
edgesfile = myworkspace+"/zh_roads.shp" #newest file!!
floodednodesfile = myworkspace+"/zh_nodes_flooded.shp"
floodededgesfile = myworkspace+"/zh_roads_flooded.shp"
floodmap = myworkspace+"/WB_HW_IK100_F_LV03dissolve.shp"

#input data: the roads file (these are two tables, geodata panda files)
nodesgdf = gpd.read_file(nodesfile)
edgesgdf = gpd.read_file(edgesfile)
floodmapgdf = gpd.read_file(floodmap)

#input data: the flooded nodes and roads files
floodednodesgdf = gpd.read_file(floodednodesfile)
floodededgesgdf = gpd.read_file(floodededgesfile)

#output data: the nodes distances file
nodesbetweennesscentralityfile=open(myworkspace+"/betweennesscentrality_floodsituation.csv","w") #developed this file from scratsch in mode w = wide.
nodesbetweennesscentralityfile.write("nodeid"+";"+"betweennesscentrality"+"\n") #header of the developed file; first column nodeid and second one "betweenesscentrality" = results of our computations

#create lists of flooded nodes and flooded edges 
listoffloodednodes=floodednodesgdf["nodeid"].unique().tolist()
listoffloodededges=floodededgesgdf["ID_Road"].unique().tolist()
print(str(len(listoffloodednodes))+" nodes are flooded")
print(str(len(listoffloodededges))+" road segments are flooded")

#create graph
G = nx.Graph()
nodesidlist=[]
edgesidlist=[]

#loop through the road shapefile to eliminate all the flooded nodes and edges:
counter=0
for index, row in edgesgdf.iterrows():
    if row.ID_Road not in listoffloodededges:
        #print(counter)
        length = row.SHAPE_Leng
        nodeid1 = row.nodeid1
        nodeid2 = row.nodeid2
        if row.nodeid1 not in listoffloodednodes:
            xcoord = nodesgdf[nodesgdf["nodeid"]==row.nodeid1].x
            ycoord = nodesgdf[nodesgdf["nodeid"] == row.nodeid1].y
            if row.nodeid1 not in G:
                G.add_node(row.nodeid1, pos=(xcoord, ycoord))
                nodesidlist.append(row.nodeid1)
        if row.nodeid2 not in listoffloodednodes:
            xcoord=nodesgdf[nodesgdf["nodeid"]==row.nodeid2].x
            ycoord = nodesgdf[nodesgdf["nodeid"] == row.nodeid2].y
            if row.nodeid2 not in G:
                G.add_node(row.nodeid2, pos=(xcoord, ycoord))
                nodesidlist.append(row.nodeid2)
        edgesidlist.append(row.ID_Road)
        if row.nodeid1 not in listoffloodednodes and row.nodeid2 not in listoffloodednodes:
            G.add_edge(row.nodeid1, row.nodeid2, weight=length)
        counter+=1
print("network graph created ...")


#calculate betweenness centrality for all nodes and write it to the output file
#parameter k is the number of the sample to safe time, k=1000 --> ca. 1% of the total network is taken as a sample. If k=None, the full network will be considered. 

betweennesscentrality=nx.betweenness_centrality(G, k=1000, normalized=True, endpoints=True) #output is a dictonary (list of number/key and a value = betweenesscentrality); endpoints have bc-value of zero, but wanted full list of nodes.

#betweennesscentrality=nx.betweenness_centrality(G, k=None, normalized=True, endpoints=True); n = key in this dictionary (=nodeID); not possible to combine number and strings, so the number has to be changed first.
for n in betweennesscentrality:
    nodesbetweennesscentralityfile.write(str(n)+";"+str(betweennesscentrality[n])+"\n")
nodesbetweennesscentralityfile.close()
print("betweenness centrality for nodes in ZH traffic network computed and exported to file ...")
