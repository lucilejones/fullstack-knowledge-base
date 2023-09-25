# Installing and using TypeScript
npm install typescript
<!-- this will install just in a specific project -->


first need npm init -y to create the package.json file 
then can install dependencies


then run the compiler
npx tsc <file name>
the compilation step will alert us to type errors and the types will be removed so just the JS runs in the browser
It will create a <file name>.js file based on the TS file

we can adjust settings in the tsconfig.json file

null and undefined are used in a specific way


# question:
one video said using inference is good and prevants redundancy, but one of the articles said not to rely on it. Which is the better practice?
