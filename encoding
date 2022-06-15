/*Karen Munyan and Candice Murray
 * CS 106B Spring 2015
 * Problem Set 6: Huffman Encoding
 * Sources: C++ reference, Stanford C++ library, lecture notes, Aubrey Colter
 *
 * This file enables encoding and compressing files using the Huffman method.
 * */

#include "encoding.h"
#include "HuffmanNode.h"
#include "bitstream.h"
#include "simpio.h"
#include "filelib.h"
#include "pqueue.h"
#include "string.h"
#include "strlib.h"
#include <iostream>

//Function Prototypes for helpers
void buildEncodingMapHelper(Map<int, string>& encodingMap, HuffmanNode* node, string code);
/*This function is a helper for building the encoding map. It takes extra parameters of the map that is being built
 * and the binary code that it is in the process of finding.*/
void decodeDataHelper(ibitstream& input, HuffmanNode* node, HuffmanNode* root, ostream& output);
/*This function is a helper for decoding the data.  It takes an extra parameter of the tree root.*/


Map<int, int> buildFrequencyTable(istream& input) {
    /*This function builds the frequency table that is the basis for the tree that will be built. */
    Map<int, int> freqTable;
    while (!input.fail()){
        int byte = input.get();
        if (byte == -1){ //if the end of the file is reached
            byte = PSEUDO_EOF;
        }
        freqTable[byte]++;
    }
    rewindStream(input);
    return freqTable;
}

HuffmanNode* buildEncodingTree(const Map<int, int>& freqTable) {
    /*This function builds the encoding tree by pulling characters out of the priority queue
     * based on the frequency with which they occurr.*/
    PriorityQueue<HuffmanNode*> queue;
    for (int key:freqTable){
        HuffmanNode* node = new HuffmanNode(key,freqTable[key]);
        queue.enqueue(node,freqTable[key]);
    }
    while(queue.size() > 1){
        HuffmanNode* node1 = queue.dequeue();
        HuffmanNode* node2 = queue.dequeue();
        int newCount = node1->count+node2->count;
        HuffmanNode* parent = new HuffmanNode(NOT_A_CHAR,newCount,node1,node2);
        queue.enqueue(parent,newCount);
    }
    return queue.dequeue();
}


Map<int, string> buildEncodingMap(HuffmanNode* encodingTree) {
    /*This function builds the map to encode files based on the encoding tree.*/
    Map<int, string> encodingMap;
    buildEncodingMapHelper(encodingMap, encodingTree,"");
    return encodingMap;
}


void buildEncodingMapHelper(Map<int, string>& encodingMap, HuffmanNode* node,string code = "")
    /*This is a helper function that does most of the work for building the encoding map.
     * It follows the tree down to each leaf, and returns the code of the path taken to get to
     * that leaf.*/
    {
    if(node == NULL)
    {
        //If the node doesn't exist, do nothing.
        return;
    }
    else if(node->zero == NULL && node->one == NULL) //if it is a leaf, return its code
    {
       encodingMap[node->character] = code;
       code = "";
       return;
    }
    else{
        code += '0'; //choose
        buildEncodingMapHelper(encodingMap, node->zero,code); //explore
        code.erase(code.length()-1,1); //unchoose
        code += '1'; //choose
        buildEncodingMapHelper(encodingMap,node->one,code); //explore
        code.erase(code.length()-1,1); //unchoose
    }
}


void encodeData(istream& input, const Map<int, string>& encodingMap, obitstream& output) {
    /*This function encodes the data from regular characters to binary Huffman encodings.*/
    while (!input.fail()){ //write the data from the file
        string bit = encodingMap[input.get()];
        for (int i = 0; i < bit.length(); i++){
            output.writeBit(bit[i]-'0');
        }
    }
    for (int i = 0; i < encodingMap[PSEUDO_EOF].length(); i++){//write the EOF code
        output.writeBit(encodingMap[PSEUDO_EOF][i]-'0');
    }
    rewindStream(input);
}

void decodeData(ibitstream& input, HuffmanNode* encodingTree, ostream& output) {
    /*This function decodes the data by calling a helper function with an additional parameter
     * of the root of the tree.*/
    decodeDataHelper(input,encodingTree,encodingTree,output);
}

void decodeDataHelper(ibitstream& input, HuffmanNode* node, HuffmanNode* root, ostream& output)
    /*This helper function does the real work in decoding the data.  It reads each bit from the input
     * and traces down the tree accordingly until it finds a leaf, then outputs that leaf's character.
     * It then starts over from the root to trace out the next character.*/
{
    if(input.fail())
    {
        //if there is no input, do nothing
        return;
    }
    else if(node->zero == NULL && node->one == NULL)
    {
        //it is a leaf, so it can return the character at that leaf
        //or the character that the code represents
        if (node->character == PSEUDO_EOF){
            return; //Reached the end of the file, don't read any more bits.
        }else{
        output.put(node->character);
        node = root;
        decodeDataHelper(input, node, root, output);//start from the root again
        return;
        }
    }
    else
    {
        int bit = input.readBit(); //reads a single bit (0 or 1) from the input
        if(bit == 0)
        {
            decodeDataHelper(input, node->zero, root, output);
        }
        else if(bit == 1)
        {
            decodeDataHelper(input, node->one, root, output);
        }
        else
        {
            return;
        }
    }
}

void compress(istream& input, obitstream& output) {
    /*This function combines steps 1-4 in order to compress a file.*/
    Map<int,int> table = buildFrequencyTable(input);
    HuffmanNode* root = buildEncodingTree(table);
    Map<int,string> codeMap = buildEncodingMap(root);
    string header = table.toString();
    for (char byte : header){//includes the frequency table as a header so that the file can be decoded
        output.put(byte);
    }
    encodeData(input,codeMap,output);
    freeTree(root);
}


void decompress(ibitstream& input, ostream& output) {
    /* Reads the frequency table Map, then decodes the rest of the data.
     */
    Map<int, int> freqTable;
    input >> freqTable;
    HuffmanNode* root = buildEncodingTree(freqTable); //build the tree with the header data
    decodeData(input, root, output);
    freeTree(root);
}


void freeTree(HuffmanNode* node) {
    /*This function frees the memory taken up by the encoding tree.*/
    if(node == NULL)
    {
        return;
    }
    else if(node->zero == NULL && node->one == NULL)
    {
        HuffmanNode* leafTrash = node;
        delete leafTrash;
    }
    else
    {
        freeTree(node->zero);
        freeTree(node->one);
    }
}
