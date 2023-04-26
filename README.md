# complexity-logger-method-
#The code for add a logger method to the complexity project and parameter using parseargs that sets nmax is:

'''

@author: programing
'''
from timeit import default_timer as timer 
from argparse import ArgumentParser 
from argparse import RawDescriptionHelpFormatter 
from datetime import datetime

import unittest 
import sys 
import os 
import hashlib
from pathlib import Path

import logging
logger = logging.getLogger(__name__) 


def initLogger(md5string):

    now = datetime.now()
    dt_string = now.strftime("%Y%m%d_%H%M%S")
    logFolder = os.path.join(os.getcwd(), "logfiles")
    if not os.path.exists(logFolder):
        print("--log folder <" + logFolder + "> doesn't exist, creating")
        os.makedirs(logFolder)
    logfileName = os.path.join(logFolder, "__" + dt_string + "__" + md5string +".log")
    handler = logging.StreamHandler(sys.stdout)
    logging.basicConfig(level=logging.DEBUG)

    fileh = logging.FileHandler(logfileName, 'a')
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    fileh.setFormatter(formatter)

    log = logging.getLogger()  # root logger
    log.setLevel(logging.DEBUG)
    for hdlr in log.handlers[:]:  # remove all old handlers
        log.removeHandler(hdlr)
    log.addHandler(fileh)      # set the new handler
    log.addHandler(handler)
    logging.info("+" + "*"*78 + "+")
    logging.info("project log file is <" + logfileName + ">")
    logging.info("+" + "*"*78 + "+")
    logging.debug("debug mode is on")


def parseArgs():
    '''parse out Command line options.'''
    try:
        parser = ArgumentParser(description="a program to calculate Fibonacci's number", formatter_class=RawDescriptionHelpFormatter)
        parser.add_argument("-m", "--max_number", dest="maxnumber", action="store", help="max value to calculate Fibonacci number [default: %(default)s]")
        parser.add_argument("-n", "--nmax", dest="nmax", action="store", help="max number of Fibonacci to be calculated [default: %(default)s]")
        
        # Process arguments
        args = parser.parse_args()
        global maxNumber, nmax
        maxNumber = args.maxnumber
        nmax = int(args.nmax) if args.nmax else 20
        
     # Set maxNumber to 10 if not specified
        if not maxNumber:
            maxNumber = 20
        # check the user specified a max number of Fibonacci, if not warn and and exit
        if maxNumber:
            logger.info("max Fibonacci number to calculate is <" + str(maxNumber) + ">")
        else:
            logger.error("you must specify a max Fibonacci number")
            sys.exit(1)
    except KeyboardInterrupt:
        ### handle keyboard interrupt ###
        return 0
    except Exception as e:
        logger.error(e) 
def timeSimpleVersusDynamicFibo(nMax):
    '''
    compare times for simple and dynamic methods to calculate Fibonacci's number
    return fibonacciTimes
    '''
    n=0
    fibonacciTimes=[]
    while n<nMax:
        logger.debug("--- " + str(n))
        simpleStartTime = timer()
        fibSimple(n)
        simpleEndTime = timer()
        dynamicStartTime = timer()
        fibDynamic(n)
        dynamicEndTime = timer()
        fibonacciTimes.append({"n":n, "simple": (simpleEndTime-simpleStartTime), "dynamic": (dynamicEndTime-dynamicStartTime) })
        n+=1
    return fibonacciTimes
def generateTimingPlot(fibonacciTimes, nmax):
    import seaborn as sns
    import pandas as pd
    import matplotlib.pyplot as plt
    dfFibonacciTimes = pd.DataFrame(fibonacciTimes)
    dfMelt = dfFibonacciTimes.melt(id_vars=['n'], value_vars=['simple','dynamic'])
    #sns.lineplot(data=dfMelt, x="n", y="simple", hue='variable')
    plt.xlabel("Fibonacci number")
    plt.ylabel("runtime (s)",)
    sns.scatterplot(data=dfMelt, x="n", y="value", hue='variable').set(title='Title of Plot')
    timingPlotFile = os.path.join(os.getcwd(), "fibonacci_timing_0_to_" + str(nmax) + ".png")
    plt.savefig(timingPlotFile) 
def fibSimple(n):
    # base case
    if n == 0:
        return(0)
    if n == 1:
        return(1)
    # pattern: ith = (i-1)th + (i-2)th
    return(fibSimple(n-1) + fibSimple(n-2)) 
def writeTimingDataToFile(fiboTimes, nmax):  
    import csv

    timingDataFile = os.path.join(os.getcwd(), "fibonacci_timing_0_to_" + str(nmax) + ".tsv")
    print("timing data will be written to <" + timingDataFile + ">")
    with open(timingDataFile, 'w') as file:
        writer = csv.writer(file)
        writer.writerow(['n', 'recursive', 'dynamic'])
        for timing in fiboTimes:
            writer.writerow(timing.values())    
    
    print("---done")
    
def fibDynamic(n):

    
    # base case
    if n == 0:
        return(0)
    if n == 1:
        savedFibNumbers[1] = 1
        return(1)
    
    if savedFibNumbers[n] != 0:
        return savedFibNumbers[n]
    
    savedFibNumbers[n] = fibDynamic(n -1) + fibDynamic(n - 2)

    return(savedFibNumbers[n])    
 
class CheckFibonacciDynamic(unittest.TestCase):
    
    # check the fibonacci dynamic calculation is correct
    # can read more about unittesting here 
    #  https://machinelearningmastery.com/a-gentle-introduction-to-unit-testing-in-python/
    def test_negative(self):
        
        nmax= 20
        global savedFibNumbers
        savedFibNumbers=[0]*nmax
        
        message = "fibonacci calculation is wrong !"
        
        # assertEqual() to check equality of first & second value

        self.assertEqual(fibDynamic(20), 50, message)
  
        

def main(argv=None): 

    if argv is None:
        argv = sys.argv
    nmax = 20
    
    parseArgs()
    
    global savedFibNumbers
    savedFibNumbers=[0]*nmax
    

    #unittest.CheckFibonacciDynamic()
    
    fibonacciTimes = timeSimpleVersusDynamicFibo(nmax)
    writeTimingDataToFile(fibonacciTimes, nmax)
    generateTimingPlot(fibonacciTimes, nmax)
           
    md5String = hashlib.md5(b"CBGAMGOUS").hexdigest()
    parseArgs()
    
    initLogger(md5String)
    
if __name__ == '__main__':

    sys.exit(main())

#Output of logger method in fibonacci = 
timing data will be written to <C:\Users\user\programing\day6\fibonacci_timing_0_to_20.tsv>
---done

![image](https://user-images.githubusercontent.com/130226215/234343099-6c0a87ae-6618-4808-be78-73708a3846c1.png)


##Logger method for geenerating logo plots and calculate GC frequency for all miRNAs for mouse, human and rat in mature.fa
code for:
# ''' That is:
#     -f "C:\Users\THIS-PC\AP\day4\data\mature.fa" -s hsa
#     -f "C:\Users\THIS-PC\AP\day4\data\mature.fa" -s mmu
#     -f "C:\Users\THIS-PC\AP\day4\data\mature.fa" -s rno
#  '''

import json
import sys
import os
from locale import atof, setlocale, LC_NUMERIC
from datetime import datetime
import hashlib
import logging


from pathlib import Path

from argparse import ArgumentParser
from argparse import RawDescriptionHelpFormatter

__all__ = []
__version__ = 0.1
__date__ = '2023-04-11'
__updated__ = '2022-04-11'

import sequence

DEBUG = 1
TESTRUN = 0
PROFILE = 0


def initLogger(md5string):

    ''' setup log file based on project name'''
    projectBaseName = ""

    projectBaseName = Path(fastaFile).stem

    now = datetime.now()
    dt_string = now.strftime("%Y%m%d_%H%M%S")
    logFolder = os.path.join(os.getcwd(), "logfiles")
    if not os.path.exists(logFolder):
        print("--log folder <" + logFolder + "> doesn't exist, creating")
        os.makedirs(logFolder)
    logfileName = os.path.join(logFolder, projectBaseName + "__" + dt_string + "__" + md5string +".log")
    handler = logging.StreamHandler(sys.stdout)
    logging.basicConfig(level=logging.DEBUG)

    fileh = logging.FileHandler(logfileName, 'a')
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    fileh.setFormatter(formatter)

    log = logging.getLogger()  # root logger
    log.setLevel(logging.DEBUG)
    for hdlr in log.handlers[:]:  # remove all old handlers
        log.removeHandler(hdlr)
    log.addHandler(fileh)      # set the new handler
    log.addHandler(handler)
    logging.info("+" + "*"*78 + "+")
    logging.info("project log file is <" + logfileName + ">")
    logging.info("+" + "*"*78 + "+")
    logging.debug("debug mode is on")

def parseArgs(argv):
    '''parse out Command line options.'''

    program_name = os.path.basename(sys.argv[0])
    program_version = "v%s" % __version__
    program_build_date = str(__updated__)
    program_version_message = '%%(prog)s %s (%s)' % (program_version, program_build_date)
    #program_shortdesc = __import__('__main__').__doc__.split("\n")[1]
    program_license = '''%s
    i
      Created by Simon Rayner on %s.
      Copyright 2023 Oslo University Hospital. All rights reserved.
    
      Licensed under the Apache License 2.0
      http://www.apache.org/licenses/LICENSE-2.0
    
      Distributed on an "AS IS" basis without warranties
      or conditions of any kind, either express or implied.
    
    USAGE
    ''' % (program_name, str(__date__))

    try:
        # Setup argument parser
        parser = ArgumentParser(description=program_license, formatter_class=RawDescriptionHelpFormatter)
        parser.add_argument("-f", "--fasta_file", dest="fastafile", action="store", help="fasta file for which you want to calc GC% [default: %(default)s]")
        parser.add_argument("-s", "--species_code", dest="speciescode", action="store", help="three character species code [default: %(default)s]")

        # Process arguments
        args = parser.parse_args()

        global fastaFile
        global speciesCode

        fastaFile = args.fastafile
        speciesCode = args.speciescode

        if fastaFile:
            print("fasta file is <" + fastaFile + ">")

        if speciesCode:
            print("speciesCode is <" + speciesCode + ">")

    except KeyboardInterrupt:
        ### handle keyboard interrupt ###
        return 0
    except Exception as e:
        print(e)
        if DEBUG or TESTRUN:
            raise(e)
        indent = len(program_name) * " "
        sys.stderr.write(program_name + ": " + repr(e) + "\n")
        sys.stderr.write(indent + "  for help use --help")
        return 2


def calcAverageGCPercent():
    '''
    calculate GC percent for each sequence and return the average value
    :return:
    '''
    totalGCPercent = 0
    sCount = 0
    for seqLine in sequenceLines:
        seq = sequence.Sequence(headerLines[sCount], seqLine)
        seq.calcGC()
        print("for sequence <" + seq.getHeaderLine() + "> GC% is <" + str(100.0*seq.getGCPercent()) + ">")
        totalGCPercent = totalGCPercent + seq.getGCPercent()
    return totalGCPercent/len(sequenceLines)

def readFastaFile(filename):
    '''
    load specified fasta file
    :param self:
    :return:
    '''
    global headerLines    
    global sequenceLines

    # load the fasta lines into a list
    try:
        fFA = open(filename, 'r')
        fastaLines = fFA.readlines()
        fFA.close()
    except Exception as e:
        raise(e)

    headerLines = []
    headerLine = ""
    sequenceLines = []
    sequence = ""

    s = 0
    for fastaLine in fastaLines:
        if fastaLine[0] == '>':
            if s > 0 and headerLine.startswith(speciesCode):
                headerLines.append(headerLine)
                sequenceLines.append(sequence)
                sequence = ""
            headerLine = fastaLine[1:].strip()
        else:
            sequence = sequence + fastaLine.strip()
        s += 1
    if headerLine.startswith(speciesCode):
        headerLines.append(headerLine)
        sequenceLines.append(sequence)
    return len(headerLines)


def main(argv=None): # IGNORE:C0111

    #setlocale(LC_NUMERIC, 'no_NO')
    if argv is None:
        argv = sys.argv

    #md5String = hashlib.md5(b"CBGAMGOUS").hexdigest()
    parseArgs(argv)
    #initLogger(md5String)
    n = readFastaFile(fastaFile)
    print("found <" + str(n) + "> sequences")

    avGCPercent = calcAverageGCPercent()
    print("average GC % = <" + str(100.0*avGCPercent) + ">")

    global filename

    filename = r"C:\Users\user\programing\shuffle\mature.fa"
    
    #n = readFastaFile(filename)

    print ("The number of sequences are: ", str(n))
    
    md5String = hashlib.md5(b"CBGAMGOUS").hexdigest()
    parseArgs(argv)
    
    initLogger(md5String)    
     

    
if __name__ == '__main__':

    sys.exit(main())

out for hsa:

average GC % = <70.05512368618675>
The number of sequences are:  2656
fasta file is <C:\Users\user\programing\shuffle\mature.fa>
speciesCode is <hsa>
+******************************************************************************+
project log file is <C:\Users\user\programing\day6\logfiles\mature__20230425_174601__3697017c62772d71b4445895933026a4.log>
+******************************************************************************+
debug mode is on
#mature_uniqseeds are 2094
 #mature_uniqseeds_logoplt
  ![image](https://user-images.githubusercontent.com/130226215/234347116-efbd2ae5-7478-4278-8a38-9f2fd8defeda.png)


#mmu logoplt
    number of mature_uniquseeds=1588
average GC % = <70.50310821391368>
The number of sequences are:  1978
fasta file is <C:\Users\user\programing\shuffle\mature.fa>
speciesCode is <mmu>
+******************************************************************************+
project log file is <C:\Users\user\programing\day6\logfiles\mature__20230425_180631__3697017c62772d71b4445895933026a4.log>
+******************************************************************************+
debug mode is on
    ![image](https://user-images.githubusercontent.com/130226215/234352020-e5717e32-6c12-4fe8-b803-33c0e2f6ea2e.png)
##rno logo plot
    number of mature_uniquseeds=635
    
    average GC % = <67.4683526671524>
The number of sequences are:  764
fasta file is <C:\Users\user\programing\shuffle\mature.fa>
speciesCode is <rno>
+******************************************************************************+
project log file is <C:\Users\user\programing\day6\logfiles\mature__20230425_181347__3697017c62772d71b4445895933026a4.log>
+******************************************************************************+
debug mode is on
#mature_uniqseeds_logoplt=
![image](https://user-images.githubusercontent.com/130226215/234353226-0255825b-b35e-4860-901f-30a82b2204b6.png)
    




