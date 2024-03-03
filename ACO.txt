import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D


#step-2
def setup_location_coordinates(number_of_locations):
    location_coordinates = np.random.rand(number_of_locations, 3)
    print(number_of_locations, "\nrandom location generated, the coordinates are as follows: ")
    print(location_coordinates)
    print(type(location_coordinates[0][0]))
    return location_coordinates
    

#step-3
number_of_locations = 30
location_coordinates = setup_location_coordinates(number_of_locations)


#step-3*
def visualise_3d_coordinates(location_coordinates, provided_path=None):
    fig = plt.figure(figsize=(8, 6))
    ax = fig.add_subplot(111, projection='3d')
    ax.scatter(location_coordinates[:, 0], location_coordinates[:, 1], location_coordinates[:, 2], c='r', marker='o')
    if provided_path:
        plot_3d_path(ax, provided_path)
    ax.set_xlabel('X Label')
    ax.set_ylabel('Y Label')
    ax.set_zlabel('Z Label')
    plt.show()

visualise_3d_coordinates(location_coordinates)



#step-4
def plot_3d_path(ax, provided_path):
    for i in range(len(provided_path)-1):
        #plot in line segment between curent plot and next plot
        ax.plot([location_coordinates[provided_path[i], 0], location_coordinates[provided_path[i+1], 0]],
                [location_coordinates[provided_path[i], 1], location_coordinates[provided_path[i+1], 1]],
                [location_coordinates[provided_path[i], 2], location_coordinates[provided_path[i+1], 2]],
                c='g', linestyle='-', linewidth=2)

        #connect the last point to the first point to complete the path
        ax.plot([location_coordinates[provided_path[0], 0], location_coordinates[provided_path[-1], 0]],
                [location_coordinates[provided_path[0], 1], location_coordinates[provided_path[-1], 1]],
                [location_coordinates[provided_path[0], 2], location_coordinates[provided_path[-1], 2]],
                c='g', linestyle='-', linewidth=2)



#step-5

default_path = []
for i in range(len(location_coordinates)):
    default_path.append(i)


visualise_3d_coordinates(location_coordinates, default_path)    

        



#step-6
number_of_ants = 50
number_of_iterations = 10
alpha = 1
beta = 1
evaporation_rate = 0.5
pheromone_increment = 1





#step-7
def ant_travel(location_coordinates, pheromone_matrix):
    #setup ant so the path and path length can be stored
    visited_locations = [False]*len(location_coordinates)
    current_location = np.random.randint(len(location_coordinates))
    visited_locations[current_location] = True
    path = [current_location]
    path_length = 0

    print(current_location)
    print(visited_locations)

    #step-10*
    while False in visited_locations:
        unvisited = np.where(np.logical_not(visited_locations))[0]
        print("\nUnvisited", unvisited)


        ## INSERT CODE HERE LATER STEP
        #step-15
        probabilities = np.zeros(len(unvisited))
        print("probabilities initialised:")
        print(probabilities)

        probabilites = calculate_probabilities(unvisited, current_location, alpha, beta, probabilities, location_coordinates, pheromone_matrix)
        print("probabilities initialised:")
        print(probabilities)
        print("sum of probabilities calculated: ", np.sum(probabilities))

        probabilities = probabilities / np.sum(probabilities)
        print("probabilities calculated post normalisation")
        print(probabilities)
        print("sum of probabilities calculated post normalisation: ", np.sum(probabilities))

        next_point = np.random.choice(unvisited, p=probabilities)   # MODIFY LINE LATER STEP
        path.append(next_point)
        path_length += distance(location_coordinates[current_location], location_coordinates[next_point])
        visited_locations[next_point] = True
        current_location = next_point


    return path, path_length




def ant_colony_optimisation(location_coordinates, number_of_ants, number_of_iterations, alpha, beta, evaporation_rate, pheromone_increment):
    # initialize best path, length
    best_path = None
    best_path_length = float('inf')
    pheromone_matrix = np.ones((number_of_locations, number_of_locations))
    print("\nPheromone matrix initialised")
    print(pheromone_matrix)
    
    #step-8
    for iteration in range(number_of_iterations):
        paths = []
        path_lengths = []

        # create a for lop to perform the actions of each ant in the colony
        for ant in range(number_of_ants):

            #create an individual ant path list and call a function that makes the ant move through the locations
            ants_path_values = ant_travel(location_coordinates, pheromone_matrix)
            paths.append(ants_path_values[0])
            path_lengths.append(ants_path_values[1])



            #step-11
            if path_lengths[ant] < best_path_length:
                best_path = paths[ant]
                best_path_length = path_lengths[ant]
                print("\nNew shortest path found in iteration:", iteration, "by ant:", ant, "with a path distance of:", best_path_length)
                print("\nNew best path is as follows:")
                print(best_path)

    #step-13*
    print("Pheromone_matrix - pre-evaporation: ", np.sum(pheromone_matrix))
    print(pheromone_matrix)
    pheromone_matrix *= evaporation_rate
    print("Pheromone_matrix - post-evaporation / pre-update: ", np.sum(pheromone_matrix))
    pheromone_matrix = lay_pheramone(paths, path_lengths, pheromone_increment, pheromone_matrix)
    print("Pheromone_matrix matrix post-update ", np.sum(pheromone_matrix))
    print(pheromone_matrix)
    
    
    
    # visualise the best path at the end
    visualise_3d_coordinates(location_coordinates, best_path)

    # print the best path
    print("\nBest path found has a distabnce of :", best_path_length, "with the following order:")
    print(best_path)


    


#step-9
def distance(start_point, end_point):
    distance_between_points = np.sqrt(np.sum((start_point - end_point)**2))
    print("Distance between:",start_point,"and",end_point,"=", distance_between_points)
    return distance_between_points



#step-12
def lay_pheramone(paths, path_lengths, pheromone_increment, pheromone_matrix):
    for ant_path, ant_path_length in zip(paths, path_lengths):
        for i in range(number_of_locations-1):
            pheromone_matrix[ant_path[i], ant_path[i+1]] += pheromone_increment/ant_path_length
        pheromone_matrix[ant_path[-1], ant_path[0]] += pheromone_increment/ant_path_length
    return pheromone_matrix

    
#step-14
def calculate_probabilities(unvisited, current_location, alpha, beta, probabilities, location_coordinates, pheromone_matrix):
    for i, unvisited_point in enumerate(unvisited):
        probabilities[i] = pheromone_matrix[current_location, unvisited_point]**alpha / distance(location_coordinates[current_location], location_coordinates[unvisited_point])**beta
    return probabilities


#keep this call at the end of the code
ant_colony_optimisation(location_coordinates, number_of_ants, number_of_iterations, alpha, beta, evaporation_rate, pheromone_increment)
    

    
































        

        
    
