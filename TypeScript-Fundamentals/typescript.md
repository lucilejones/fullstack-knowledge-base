# Installing and using TypeScript
TS docs: https://www.typescriptlang.org/


first need npm init -y to create the package.json file 
then can install dependencies

to install TS per project: 
npm install typescript --save-dev

tp install TS globally:
npm install -g typescript

then run the compiler
npx tsc <file name>.ts
the compilation step will alert us to type errors and the types will be removed so just the JS runs in the browser
It will create a <file name>.js file based on the TS file
Then we can do node <file name>.js

To compile all the files at once we run npx tsc --init

we can adjust settings in the tsconfig.json file




# question:
one video said using inference is good and prevants redundancy, but one of the articles said not to rely on it. Which is the better practice?
