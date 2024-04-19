import os, sys
import numpy as np

def line2list(str_data, start_col, cols):
    temp_list = [0.0 for i in range(cols)]
    str_data = str_data.split(',')
    num_data = str_data[start_col:]
    end_i = 0
    for i, d in enumerate(num_data):
        if d != '':
            temp_list[i] = float(d)
        else:
            temp_list[i] = "nop"
        end_i = i
    for i in range(end_i, cols):
        temp_list[i] = "nop"

    return temp_list

def get_data(file_path, start_col, cols):
    temp_data = [[] for i in range(cols)]

    with open(file_path) as f:
        while True:
            line = f.readline()
            if line[:7] == "\"Frame\"" or len(line) < 1:
                break
            if line[:5] == "Frame" or len(line) < 1 :
                break
        while True:
            line = f.readline()
            if len(line) < 1:
                break
            line = line[0:-1]
            line_data =line2list(line, start_col, cols)

            for i, d in enumerate(line_data):
                if d != "nop":
                    temp_data[i].append(d)
    
    return temp_data

def remove_negative_row(data_list):
    new_list = []
    for d_l in data_list:
        if len(d_l) > 0:
            if np.mean(d_l) > 0:
                new_list.append(d_l)
            else:
                new_list.append([])
        else:
            new_list.append([])
    
    return new_list

def remove_negative_block(data_list, negative_times):
    negative_num = 0
    negative_list = []
    new_data_list = []
    for d in data_list:
        if d < 0:
            negative_num += 1
            if negative_num >= negative_times:
                negative_list.append(2)
            else:
                negative_list.append(1)
        else:
            negative_num = 0
            negative_list.append(0)

    flag = 0
    for i in range(len(data_list) - 1, -1, -1):
        if flag == 1 and negative_list[i] != 0:
            negative_list[i] = 2
        else:
            flag = 0

        if negative_list[i] == 2:
            flag = 1

        if negative_list[i] == 1:
            negative_list[i] = 0

    for i, d in enumerate(data_list):
        if negative_list[i] == 0:
            new_data_list.append(d)
    
    return new_data_list

def get_mean(remove_negative_data_list, len_data):
    mean_data = []
    for d in remove_negative_data_list:
        if len(d) > len_data:
            mean_data.append(np.mean(d))
        else:
            mean_data.append(0.0)
    return mean_data

def get_cols(file_path):
    cols = 0
    with open(file_path) as f:
        while True:
            line = f.readline()
            if line[:7] == "\"Frame\"":
                cols = int(line.split("\"")[-2])
                print(cols)
                break
            if line[:5] == "Frame":
                cols = int(line.split(",")[-1])
                print(cols)
                break
    return cols

# file_path = "Speed-2.csv"
# outfile_path = "Speed-2.txt"
file_path = sys.argv[1]
outfile_path = sys.argv[2]
cols = get_cols(file_path)
start_col = 2
negative_times = 2
len_data = 100
data = get_data(file_path, start_col, cols)
data = remove_negative_row(data)
remove_negative_data = [remove_negative_block(d, negative_times) for d in data]
mean_data = get_mean(remove_negative_data, len_data)

with open(outfile_path, 'w') as f:
    for i, d in enumerate(mean_data):
        f.write("{}, {}".format(i, d))
        f.write("\n")
print("Operation to complete!")
![image](https://github.com/lingling-ll/Analyzing-speed-of-C.elegans/assets/167515121/3d1b9612-63b1-4e98-b7db-50794bc8a9b5)
