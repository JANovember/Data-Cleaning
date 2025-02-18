``` python
import csv

def chunck_csv(filename):
    try:
        with open(filename, 'r') as f:
            reader = csv.reader(f)
            data = list(reader)
            chunk_size = len(data) // 10
            # split the data into 10 chunks of roughly equal size
            # for example, if data has 100 items, chunks will be
            # [[item1,item2,...,item10], [item11,item12,...,item20], ...]
            chunks = [data[i:i+chunk_size] for i in range(0, len(data), chunk_size)]
            for i, chunk in enumerate(chunks):
                with open(filename + '_' + str(i) + '.csv', 'w', newline='') as f_out:
                    writer = csv.writer(f_out)
                    writer.writerows(chunk)
        print("From chunck_csv: CSV file has been successfully split into chunks.")
    except FileNotFoundError:
        print("From chunck_csv: File not found. Please check the filename.")
    except Exception as e:
        print("From chunck_csv: An error occurred:", str(e))
```
``` python
def combine_csv(filename):
    try:
        with open(filename, 'w', newline='') as f:
            writer = csv.writer(f)
            for i in range(10):
                with open(filename+'_'+str(i)+'.csv', 'r') as f2:
                    reader = csv.reader(f2)
                    writer.writerows(reader)
        print("From combine_csv: Combined csv file saved as", filename)
    except FileNotFoundError:
        print("From combine_csv: File not found. Please check the filename.")
    except Exception as e:
        print("From combine_csv: An error occurred:", str(e))
```
