
/*util.h  - Header file for implementation of functions used to compress
        and decompress files using the Huffman Encoding Algorithim

            written by: [George Fashho] on [11/24/2022] for [CS 251 FA22]
 */
#include <queue> 
#pragma once


typedef hashmap hashmapF;
typedef unordered_map <int, string> hashmapE;

struct HuffmanNode {
    int character;
    int count;
    HuffmanNode* zero;
    HuffmanNode* one;
};

struct compare
{
    bool operator()(const HuffmanNode *lhs,
        const HuffmanNode *rhs)
    {
        return lhs->count > rhs->count;
    }
};


/*Freeing memeory allocated to HuffmanNode using In-Order Traversal*/
void freeTree(HuffmanNode* node) { 
    
    if(node == nullptr){
        return;
    }    
    freeTree(node->zero);
    freeTree(node->one);

    delete node;
}


/*
* This function builds the frequency map.  If isFile is true, then it reads
* from filename.  If isFile is false, then it reads from a string filename.
*
* @param filename: string or filename to build frequency map upon
* @param isFile : Decides whether we using a string of ifstream
* @param map : hashMap we are building 
*/
void buildFrequencyMap(string filename, bool isFile, hashmapF &map) {

    if(isFile){ //checking if it is a valid filename

        ifstream inFile(filename);
        char c;

        while(inFile.get(c)){
            if(map.containsKey(c)){
                map.put(c,map.get(c)+1); //incrementing keys value if already exists
            }
            else{
                map.put(c,1); //setting value to one if key doesnt exisist
            }
        }
    }
    else{
        for(char c : filename){ //traversig every char in the string
            if(map.containsKey(c)){
                map.put(c,map.get(c)+1);
            }
            else{
                map.put(c,1);
            }
        }
    }
    map.put(PSEUDO_EOF, 1 ); //adding EOF to hashmap at the end 
}


/*
* This function builds an encoding tree from the frequency map.
*
* @param map : UnOrdered_Map we are building
*/
HuffmanNode* buildEncodingTree(hashmapF &map) {
   
    priority_queue< HuffmanNode*,vector<HuffmanNode*>,compare> pq;  //priority queue
    
    vector<int>mapKeys = map.keys() ;  //building a vector containing all the keys in the map 

    for(uint i = 0 ; i < mapKeys.size() ; i ++ ){  //looping through every key and building a node to add to priotriy queue

        HuffmanNode * temp = new HuffmanNode() ; 

        temp->character = mapKeys[i]; 
        temp->count = map.get(mapKeys[i]) ;  //initializing node
        temp->one = nullptr ; 
        temp->zero = nullptr ; 

        pq.push(temp); //pushing into queue
    }

    while(pq.size() > 1 ){ 

        HuffmanNode *newNode = new HuffmanNode(); //creating new node 

        newNode->zero = pq.top(); //left node being first in queue 
        pq.pop();   //popping off queue
        newNode->one = pq.top(); 
        pq.pop();

        newNode->count = newNode->zero->count + newNode->one->count ; //new count 
        newNode->character = NOT_A_CHAR ;

        pq.push(newNode);  
        }

    return pq.top(); //returning root node
}


/*
* Recursive helper function for building the encoding map of 0's & 1's
*
* @param node : current node 
* @param encodingMap: BST of nodes to search
* @param str : empty str to build binary code encoding
* @param prev : nullptr 
*/
void _buildEncodingMap(HuffmanNode* node, hashmapE &encodingMap, string str,
                       HuffmanNode* prev) {
    
    if(node->character != NOT_A_CHAR){  //return once NAC is reached
        encodingMap[node->character] = str;
        return;
    }

    str += "0";
    _buildEncodingMap(node->zero,encodingMap,str,nullptr); //in-order recursion 

    str.pop_back();

    str += "1";
    _buildEncodingMap(node->one,encodingMap,str,nullptr);

    str.pop_back();
}


/*
* This function builds the encoding map from an encoding tree.
*
* @param tree: BST Tree to traverse 
*/
hashmapE buildEncodingMap(HuffmanNode* tree) {
    
    hashmapE encodingMap;
    
    string encodedStr = "";

    if(tree){
        _buildEncodingMap(tree,encodingMap,encodedStr,nullptr);
    }
    
    return encodingMap; 
}


/*
* This function encodes the data in the input stream into the output stream
* using the encodingMap.  This function calculates the number of bits
* written to the output stream and sets result to the size parameter, which is
* passed by reference.  This function also returns a string representation of
* the output file, which is particularly useful for testing.
* 
* @param input: input stream
* @param encodingMap: Map of encoded characters
* @param output: output stream
* @param size: size of encoded data
* @param makeFile: boolean
*/
string encode(ifstream& input, hashmapE &encodingMap, ofbitstream& output,
              int &size, bool makeFile) {
    
    char c ;
    string binary = ""; //string to build upon

    while(input.get(c)){
          binary += encodingMap[c] ; //getting encoding for each character in the map
    }
    
    binary += encodingMap[PSEUDO_EOF]; //adding end of line

    size = binary.size();

    if(makeFile){
        for(char i : binary){ //writing 0's and 1's to output depending on binary string

            if(i =='0'){
                output.writeBit(0);
            }
            else{
                output.writeBit(1);
            }
        }
    }    
    return binary;  
}



/*
* This function decodes the input stream and writes the result to the output
* stream using the encodingTree.  This function also returns a string
* representation of the output file, which is particularly useful for testing.
*
* @param encodingTree: BST tree to traverse
*/
string decode(ifbitstream &input, HuffmanNode* encodingTree, ofstream &output) {
    
    HuffmanNode* root = encodingTree ; //root of tree

    string decodedString = "" ;

    while(!input.eof()){

        int bit = input.readBit(); //reading bits

        if(bit == 1 ){
            encodingTree = encodingTree->one; //going right if bit == 1

        }
        else{
            encodingTree = encodingTree->zero; //going left if bit == 0
        }
        
        if(encodingTree->character == PSEUDO_EOF){break;} //break loop if EOF reached
        
        if(encodingTree->character != NOT_A_CHAR){
            
            decodedString += encodingTree->character ; 

            output.put(encodingTree->character ); 
            encodingTree = root; //going back to the root of the BST
            continue;
        }
    }
    return decodedString;  //returning decoding string
}


/*
* This function completes the entire compression process.  Given a file,
* filename, this function (1) builds a frequency map; (2) builds an encoding
* tree; (3) builds an encoding map; (4) encodes the file (don't forget to
* include the frequency map in the header of the output file).  This function
* should create a compressed file named (filename + ".huf") and should also
* return a string version of the bit pattern.
*
* @param filename: name of file or str to compress
*/
string compress(string filename) {
    
    hashmapF map;
    buildFrequencyMap(filename,true,map); //(1) Building Frequency Map

    HuffmanNode* encodingTree = buildEncodingTree(map); // (2) Building Encoding Tree
    hashmapE encodingMap = buildEncodingMap(encodingTree);// (3) Bulding Encoding Map

    ofbitstream output(filename + ".huf" ); // (4) New File Name

    output<<map ;

    ifstream input(filename);

    int size = 0 ;
    string encodedString = encode(input,encodingMap,output,size,true); // (5) Encoding the file

    freeTree(encodingTree); //Freeing Memory 

    return encodedString;  
}


/*
* This function completes the entire decompression process.  Given the file,
* filename (which should end with ".huf"), (1) extract the header and build
* the frequency map; (2) build an encoding tree from the frequency map; (3)
* using the encoding tree to decode the file.  This function should create a
* compressed file using the following convention.
* If filename = "example.txt.huf", then the uncompressed file should be named
* "example_unc.txt".  The function should return a string version of the
* uncompressed file.  Note: this function should reverse what the compress
* function did.
*
* @param filename: name of file or str to compress
*/
string decompress(string filename) {
    
    string OGFileName = filename; 
    ifbitstream input(filename) ; 

    filename.erase(filename.find('.'),filename.length());
    ofstream output(filename + "_unc.txt");

    hashmapF map; 
    input >> map; //(1) Building Map off of input
    HuffmanNode* encodingTree = buildEncodingTree(map); // (2) Building Encoding Tree

    string decodedString = decode(input,encodingTree,output); // (3) Decoding File
    freeTree(encodingTree); // (5) Free Allocated Memory

    return decodedString;  
}
